<template>
  <teleport to="body">
    <div v-if="active" class="modal is-active">
      <div class="modal-background" @click="$emit('close')"></div>

      <div
        class="modal-card"
        role="dialog"
        aria-modal="true"
        :aria-label="title"
      >
        <header class="modal-card-head">
          <p class="modal-card-title">{{ title }}</p>
          <button
            class="delete"
            type="button"
            aria-label="close"
            @click="$emit('close')"
          ></button>
        </header>

        <section class="modal-card-body">
          <div class="service-modal-header">
            <div v-if="item?.logo" class="service-modal-logo">
              <img :src="item.logo" :alt="`${item?.name || 'service'} logo`" />
            </div>
            <div v-else-if="item?.icon" class="service-modal-logo">
              <i :class="['fa-fw', item.icon]" style="font-size: 40px"></i>
            </div>

            <div class="service-modal-meta">
              <p v-if="item?.subtitle" class="subtitle is-6">
                {{ item.subtitle }}
              </p>
              <p v-if="item?.tag" class="tags">
                <span class="tag" :class="item.tagstyle">#{{ item.tag }}</span>
              </p>
            </div>
          </div>

          <div class="service-modal-actions">
            <button
              class="button is-small is-light"
              type="button"
              @click="copyDetails"
            >
              <span class="icon is-small">
                <i class="fa-regular fa-copy"></i>
              </span>
              <span>{{ copied ? "Copiado" : "Copiar detalhes" }}</span>
            </button>
          </div>

          <table class="table is-fullwidth is-striped is-hoverable">
            <tbody>
              <tr>
                <th>Acesso</th>
                <td>
                  <span
                    class="tag"
                    :class="webPanelInfo.tagClass"
                  >
                    {{ webPanelInfo.label }}
                  </span>
                </td>
              </tr>
              <tr v-if="note">
                <th>Anotação</th>
                <td>{{ note }}</td>
              </tr>
              <tr v-if="item?.type">
                <th>Type</th>
                <td>{{ item.type }}</td>
              </tr>
              <tr v-if="item?.url">
                <th>URL</th>
                <td>
                  <a
                    :href="item.url"
                    :target="item.target || '_self'"
                    rel="noreferrer"
                    >{{ item.url }}</a
                  >
                </td>
              </tr>
              <tr v-if="item?.target">
                <th>Target</th>
                <td>{{ item.target }}</td>
              </tr>
              <tr v-if="item?.keywords">
                <th>Keywords</th>
                <td>{{ item.keywords }}</td>
              </tr>
              <tr v-if="item?.endpoint">
                <th>Endpoint</th>
                <td>{{ item.endpoint }}</td>
              </tr>
              <tr v-if="item?.class">
                <th>Class</th>
                <td>{{ item.class }}</td>
              </tr>
            </tbody>
          </table>

          <div
            v-if="Array.isArray(item?.quick) && item.quick.length"
            class="quicklinks-block"
          >
            <p class="title is-6">Quick links</p>
            <div class="buttons">
              <a
                v-for="(link, linkIndex) in item.quick"
                :key="linkIndex"
                class="button is-small"
                :style="
                  link?.color
                    ? `background-color:${link.color}; border-color:${link.color};`
                    : ''
                "
                :href="link.url"
                :target="link.target || '_self'"
                rel="noreferrer"
              >
                <span v-if="link.icon" class="icon">
                  <i :class="['fa-fw', link.icon]"></i>
                </span>
                <span>{{ link.name }}</span>
              </a>
            </div>
          </div>

          <details class="details-block">
            <summary>Raw item</summary>
            <pre class="raw">{{ prettyItem }}</pre>
          </details>
        </section>

        <footer class="modal-card-foot">
          <a
            v-if="item?.url"
            class="button is-primary"
            :href="item.url"
            :target="item.target || '_self'"
            rel="noreferrer"
          >
            Abrir serviço
          </a>
          <button class="button" type="button" @click="$emit('close')">
            Fechar
          </button>
        </footer>
      </div>
    </div>
  </teleport>
</template>

<script>
export default {
  name: "ServiceInfoModal",
  props: {
    item: {
      type: Object,
      default: null,
    },
    active: {
      type: Boolean,
      default: false,
    },
  },
  emits: ["close"],
  data() {
    return {
      copied: false,
      copiedTimer: null,
    };
  },
  computed: {
    title() {
      return this.item?.name
        ? `Detalhes: ${this.item.name}`
        : "Detalhes do serviço";
    },
    webPanelInfo() {
      const raw =
        this.item?.webPanel !== undefined
          ? this.item.webPanel
          : this.item?.hasWebPanel;

      const normalizeLabel = (value) => {
        const upper = String(value).trim().toUpperCase();
        return upper.length > 12 ? upper.slice(0, 12) : upper;
      };

      if (typeof raw === "string") {
        const value = raw.trim().toLowerCase();
        if (!value || value === "web") {
          return { label: "WEB", tagClass: "is-success" };
        }
        if (value === "none" || value === "no" || value === "false") {
          return { label: "NÃO", tagClass: "is-danger" };
        }
        const label = normalizeLabel(value);
        return { label, tagClass: "is-info" };
      }

      if (raw === true) return { label: "WEB", tagClass: "is-success" };
      if (raw === false) return { label: "NÃO", tagClass: "is-danger" };

      // fallback: se tiver URL, assumimos WEB
      return this.item?.url
        ? { label: "WEB", tagClass: "is-success" }
        : { label: "NÃO", tagClass: "is-danger" };
    },
    note() {
      return (
        this.item?.note || this.item?.anotacao || this.item?.annotation || ""
      );
    },
    prettyItem() {
      try {
        return JSON.stringify(this.item ?? {}, null, 2);
      } catch {
        return String(this.item);
      }
    },
    detailsText() {
      const lines = [];
      const name = this.item?.name || "(sem nome)";
      lines.push(`Serviço: ${name}`);

      if (this.item?.subtitle) lines.push(`Subtitle: ${this.item.subtitle}`);
      lines.push(`Painel web: ${this.hasWebPanel ? "Sim" : "Não"}`);
      if (this.note) lines.push(`Anotação: ${this.note}`);

      if (this.item?.url) lines.push(`URL: ${this.item.url}`);
      if (this.item?.target) lines.push(`Target: ${this.item.target}`);
      if (this.item?.type) lines.push(`Type: ${this.item.type}`);
      if (this.item?.endpoint) lines.push(`Endpoint: ${this.item.endpoint}`);
      if (this.item?.keywords) lines.push(`Keywords: ${this.item.keywords}`);
      if (this.item?.tag) lines.push(`Tag: #${this.item.tag}`);

      if (Array.isArray(this.item?.quick) && this.item.quick.length) {
        lines.push("Quick links:");
        for (const link of this.item.quick) {
          const linkName = link?.name || "(sem nome)";
          const linkUrl = link?.url || "";
          lines.push(`- ${linkName}${linkUrl ? `: ${linkUrl}` : ""}`);
        }
      }

      return lines.join("\n");
    },
  },
  watch: {
    active: {
      immediate: true,
      handler(isActive) {
        const root = document?.documentElement;
        if (!root) return;
        if (isActive) root.classList.add("is-clipped");
        else root.classList.remove("is-clipped");

        if (isActive) {
          window.addEventListener("keydown", this.onWindowKeydown);
          this.copied = false;
        } else {
          window.removeEventListener("keydown", this.onWindowKeydown);
          if (this.copiedTimer) {
            clearTimeout(this.copiedTimer);
            this.copiedTimer = null;
          }
        }
      },
    },
  },
  beforeUnmount() {
    document?.documentElement?.classList.remove("is-clipped");
    window.removeEventListener("keydown", this.onWindowKeydown);
    if (this.copiedTimer) clearTimeout(this.copiedTimer);
  },
  methods: {
    async copyDetails() {
      const text = this.detailsText;

      const setCopied = () => {
        this.copied = true;
        if (this.copiedTimer) clearTimeout(this.copiedTimer);
        this.copiedTimer = setTimeout(() => {
          this.copied = false;
          this.copiedTimer = null;
        }, 1500);
      };

      try {
        await navigator.clipboard.writeText(text);
        setCopied();
        return;
      } catch {
        // fallback
        const textarea = document.createElement("textarea");
        textarea.value = text;
        textarea.setAttribute("readonly", "");
        textarea.style.position = "fixed";
        textarea.style.left = "-9999px";
        document.body.appendChild(textarea);
        textarea.select();
        document.execCommand("copy");
        document.body.removeChild(textarea);
        setCopied();
      }
    },
    onWindowKeydown(event) {
      if (!this.active) return;
      if (event.key === "Escape") this.$emit("close");
    },
  },
};
</script>

<style scoped lang="scss">
.service-modal-header {
  display: flex;
  gap: 0.75rem;
  align-items: center;
  margin-bottom: 1rem;
}

.service-modal-logo {
  width: 56px;
  height: 56px;
  display: flex;
  align-items: center;
  justify-content: center;

  img {
    width: 56px;
    height: 56px;
    object-fit: contain;
  }
}

.service-modal-actions {
  display: flex;
  justify-content: flex-end;
  margin: -0.5rem 0 0.75rem;
}

.quicklinks-block {
  margin-top: 1rem;
}

.details-block {
  margin-top: 1rem;

  summary {
    cursor: pointer;
    user-select: none;
  }

  .raw {
    margin-top: 0.5rem;
    background: rgba(0, 0, 0, 0.05);
    padding: 0.75rem;
    border-radius: 0.5rem;
    overflow: auto;
    max-height: 40vh;
  }
}
</style>
