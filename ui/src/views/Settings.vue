<template>
  <cv-grid fullWidth>
    <cv-row>
      <div class="bx--col-lg-16 page-title">
        <h2>{{ $t("settings.title") }}</h2>
      </div>
    </cv-row>
    <cv-row v-if="error.getConfiguration">
      <cv-column>
        <NsInlineNotification
          kind="error"
          :title="$t('action.get-configuration')"
          :description="error.getConfiguration"
          :showCloseButton="false"
        />
      </cv-column>
    </cv-row>
    <cv-row>
      <cv-column>
        <cv-tile light>
          <cv-form @submit.prevent="configureModule">
            <cv-text-input
              :label="$t('settings.host_server')"
              v-model="host_server"
              placeholder="minio.mydomain.org"
              :disabled="loading.getConfiguration || loading.configureModule"
              :invalid-message="$t(error.host_server)"
              ref="host_server"
            ></cv-text-input>
            <cv-text-input
              :label="$t('settings.host_console')"
              v-model="host_console"
              placeholder="console.minio.mydomain.org"
              :disabled="loading.getConfiguration || loading.configureModule"
              :invalid-message="$t(error.host_console)"
              ref="host_console"
            ></cv-text-input>
            <cv-toggle
              value="letsEncrypt"
              :label="$t('settings.lets_encrypt')"
              v-model="lets_encrypt"
              :disabled="loading.getConfiguration || loading.configureModule"
              class="mg-bottom"
            >
              <template slot="text-left">{{
                $t("settings.disabled")
              }}</template>
              <template slot="text-right">{{
                $t("settings.enabled")
              }}</template>
             </cv-toggle>
            <cv-text-input
              :label="$t('settings.user')"
              v-model="user"
              :placeholder="$t('settings.user')"
              :disabled="loading.getConfiguration || loading.configureModule"
              :invalid-message="error.user"
              ref="user"
            ></cv-text-input>
            <cv-text-input
              :label="$t('settings.password')"
              v-model="password"
              :placeholder="$t('settings.password')"
              :disabled="loading.getConfiguration || loading.configureModule"
              :invalid-message="error.password"
              ref="password"
            ></cv-text-input>
              <cv-accordion ref="accordion">
              <cv-accordion-item :open="toggleAccordion[0]">
                <template slot="title">{{ $t("common.advanced") }}</template>
                <template slot="content">
                  <cv-text-input
                    :label="$t('settings.storage_path')"
                    v-model="storage"
                    :helper-text="$t('settings.storage_path_helper')"
                    :invalid-message="error.storage"
                :disabled="loading.getConfiguration || loading.configureModule"
                    ref="storage"
                  >
                  </cv-text-input>
                </template>
              </cv-accordion-item>
              </cv-accordion>
              <br/>
            <cv-row v-if="error.configureModule">
              <cv-column>
                <NsInlineNotification
                  kind="error"
                  :title="$t('action.configure-module')"
                  :description="error.configureModule"
                  :showCloseButton="false"
                />
              </cv-column>
            </cv-row>
            <NsButton
              kind="primary"
              :icon="Save20"
              :loading="loading.configureModule"
              :disabled="loading.getConfiguration || loading.configureModule"
              >{{ $t("settings.save") }}</NsButton
            >
          </cv-form>
        </cv-tile>
      </cv-column>
    </cv-row>
  </cv-grid>
</template>

<script>
import to from "await-to-js";
import { mapState } from "vuex";
import {
  QueryParamService,
  UtilService,
  TaskService,
  IconService,
} from "@nethserver/ns8-ui-lib";

export default {
  name: "Settings",
  mixins: [TaskService, IconService, UtilService, QueryParamService],
  pageTitle() {
    return this.$t("settings.title") + " - " + this.appName;
  },
  data() {
    return {
      q: {
        page: "settings",
      },
      urlCheckInterval: null,
      user: "minioadmin",
      password: "minioadmin",
      host_server: "",
      host_console: "",
      lets_encrypt: true,
      storage: "",
      loading: {
        getConfiguration: false,
        configureModule: false,
      },
      error: {
        getConfiguration: "",
        configureModule: "",
        host_console: "",
        host_server: "",
        user: "",
        password: "",
        lets_encrypt: "",
        storage: "",
      },
    };
  },
  computed: {
    ...mapState(["instanceName", "core", "appName"]),
  },
  beforeRouteEnter(to, from, next) {
    next((vm) => {
      vm.watchQueryData(vm);
      vm.urlCheckInterval = vm.initUrlBindingForApp(vm, vm.q.page);
    });
  },
  beforeRouteLeave(to, from, next) {
    clearInterval(this.urlCheckInterval);
    next();
  },
  created() {
    this.getConfiguration();
  },
  methods: {
    async getConfiguration() {
      this.loading.getConfiguration = true;
      this.error.getConfiguration = "";
      const taskAction = "get-configuration";

      // register to task error
      this.core.$root.$off(taskAction + "-aborted");
      this.core.$root.$once(
        taskAction + "-aborted",
        this.getConfigurationAborted
      );

      // register to task completion
      this.core.$root.$off(taskAction + "-completed");
      this.core.$root.$once(
        taskAction + "-completed",
        this.getConfigurationCompleted
      );

      const res = await to(
        this.createModuleTaskForApp(this.instanceName, {
          action: taskAction,
          extra: {
            title: this.$t("action." + taskAction),
            isNotificationHidden: true,
          },
        })
      );
      const err = res[0];

      if (err) {
        console.error(`error creating task ${taskAction}`, err);
        this.error.getConfiguration = this.getErrorMessage(err);
        this.loading.getConfiguration = false;
        return;
      }
    },
    getConfigurationAborted(taskResult, taskContext) {
      console.error(`${taskContext.action} aborted`, taskResult);
      this.error.getConfiguration = this.core.$t("error.generic_error");
      this.loading.getConfiguration = false;
    },
    getConfigurationCompleted(taskContext, taskResult) {
      this.loading.getConfiguration = false;
      const config = taskResult.output;

      this.host_server = config.host_server;
      this.host_console = config.host_console;
      this.user = config.user;
      this.password = config.password;
      this.lets_encrypt = config.lets_encrypt;
      this.storage = config.storage;

      this.focusElement("host_server");
    },
    validateConfigureModule() {
      this.clearErrors(this);
      let isValidationOk = true;

      if (!this.host_server) {
        // host_server field cannot be empty
        this.error.host_server = this.$t("common.required");

        if (isValidationOk) {
          this.focusElement("host_server");
          isValidationOk = false;
        }
      }

      if (!this.host_console) {
        // host_console field cannot be empty
        this.error.host_console = this.$t("common.required");

        if (isValidationOk) {
          this.focusElement("host_console");
          isValidationOk = false;
        }
      }

      if (this.host_console == this.host_server) {
        this.error.host_console = this.$t("settings.different");
        this.error.host_server = this.$t("settings.different");

        if (isValidationOk) {
          this.focusElement("host_console");
          isValidationOk = false;
        }
      }

      return isValidationOk;
    },
    configureModuleValidationFailed(validationErrors) {
      this.loading.configureModule = false;

      for (const validationError of validationErrors) {
        const param = validationError.parameter;

        // set i18n error message
        this.error[param] = this.$t("settings." + validationError.error);
      }
    },
    async configureModule() {
      const isValidationOk = this.validateConfigureModule();
      if (!isValidationOk) {
        return;
      }

      this.loading.configureModule = true;
      const taskAction = "configure-module";

      // register to task error
      this.core.$root.$off(taskAction + "-aborted");
      this.core.$root.$once(
        taskAction + "-aborted",
        this.configureModuleAborted
      );

      // register to task validation
      this.core.$root.$off(taskAction + "-validation-failed");
      this.core.$root.$once(
        taskAction + "-validation-failed",
        this.configureModuleValidationFailed
      );

      // register to task completion
      this.core.$root.$off(taskAction + "-completed");
      this.core.$root.$once(
        taskAction + "-completed",
        this.configureModuleCompleted
      );

      const res = await to(
        this.createModuleTaskForApp(this.instanceName, {
          action: taskAction,
          data: {
            host_server: this.host_server,
            host_console: this.host_console,
            user: this.user,
            password: this.password,
            lets_encrypt: this.lets_encrypt ? true : false,
            storage: this.storage
          },
          extra: {
            title: this.$t("settings.configure_instance", {
              instance: this.instanceName,
            }),
            description: this.$t("common.processing"),
          },
        })
      );
      const err = res[0];

      if (err) {
        console.error(`error creating task ${taskAction}`, err);
        this.error.configureModule = this.getErrorMessage(err);
        this.loading.configureModule = false;
        return;
      }
    },
    configureModuleAborted(taskResult, taskContext) {
      console.error(`${taskContext.action} aborted`, taskResult);
      this.error.configureModule = this.core.$t("error.generic_error");
      this.loading.configureModule = false;
    },
    configureModuleCompleted() {
      this.loading.configureModule = false;

      // reload configuration
      this.getConfiguration();
    },
  },
};
</script>

<style scoped lang="scss">
@import "../styles/carbon-utils";
</style>
