<template>
  <div>
    <div
      class="card service-card"
      :style="`background-color:${item.background};`"
    >
      <div v-if="canShowInfo" class="service-corner-actions">
        <button
          class="service-action-button"
          type="button"
          aria-label="Abrir detalhes do serviço"
          title="Detalhes"
          @click.stop.prevent="showInfo = true"
        >
          <i class="fa-solid fa-circle-info" aria-hidden="true"></i>
        </button>

        <button
          class="service-action-button is-copy"
          :class="{ 'is-copied': copied }"
          type="button"
          aria-label="Copiar detalhes do serviço"
          title="Copiar detalhes"
          @click.stop.prevent="copyDetails"
        >
          <i v-if="!copied" class="fa-regular fa-copy" aria-hidden="true"></i>
          <i v-else class="fa-solid fa-check" aria-hidden="true"></i>
        </button>

        <div
          class="service-panel-indicator"
          :class="`is-${webPanelInfo.status}`"
          :title="webPanelInfo.title"
          aria-hidden="true"
        >
          <template v-if="webPanelInfo.status === 'custom'">
            {{ webPanelInfo.label }}
          </template>
          <i v-else-if="webPanelInfo.status === 'yes'" class="fa-solid fa-check"></i>
          <i v-else class="fa-solid fa-xmark"></i>
        </div>
      </div>

      <a :href="item.url" :target="item.target" rel="noreferrer">
        <div class="card-content">
          <div :class="mediaClass">
            <slot name="icon">
              <div v-if="item.logo" class="media-left">
                <figure class="image is-48x48">
                  <img :src="item.logo" :alt="`${item.name} logo`" />
                </figure>
              </div>
              <div v-if="item.icon" class="media-left">
                <figure class="image is-48x48">
                  <i style="font-size: 32px" :class="['fa-fw', item.icon]"></i>
                </figure>
              </div>
            </slot>
            <div class="media-content">
              <slot name="content">
                <p class="title">{{ item.name }}</p>
                <p v-if="item.quick" class="quicklinks">
                  <a
                    v-for="(link, linkIndex) in item.quick"
                    :key="linkIndex"
                    :style="`background-color:${link.color};`"
                    :href="link.url"
                    :target="link.target"
                    rel="noreferrer"
                    @click.stop
                  >
                    <span v-if="link.icon"
                      ><i
                        style="font-size: 12px"
                        :class="['fa-fw', link.icon]"
                      ></i
                    ></span>
                    {{ link.name }}
                  </a>
                </p>
                <p v-if="item.subtitle" class="subtitle">
                  {{ item.subtitle }}
                </p>
              </slot>
            </div>
            <slot name="indicator" class="indicator"></slot>
          </div>
          <div v-if="item.tag" class="tag" :class="item.tagstyle">
            <strong class="tag-text">#{{ item.tag }}</strong>
          </div>
        </div>
      </a>

      <ServiceInfoModal
        :item="item"
        :active="showInfo"
        @close="showInfo = false"
      />
    </div>
  </div>
</template>

<script>
import ServiceInfoModal from "@/components/ServiceInfoModal.vue";

export default {
  name: "Generic",
  components: {
    ServiceInfoModal,
  },
  props: {
    item: Object,
  },
  data() {
    return {
      showInfo: false,
      copied: false,
      copiedTimer: null,
    };
  },
  computed: {
    mediaClass: function () {
      return { media: true, "no-subtitle": !this.item.subtitle };
    },
    canShowInfo() {
      return this.item?.disableInfoModal !== true;
    },
    webPanelInfo() {
      const raw =
        this.item?.webPanel !== undefined
          ? this.item.webPanel
          : this.item?.hasWebPanel;

      const normalizeLabel = (value) => {
        const upper = String(value).trim().toUpperCase();
        return upper.length > 6 ? upper.slice(0, 6) : upper;
      };

      if (typeof raw === "string") {
        const value = raw.trim().toLowerCase();
        if (!value || value === "web") {
          return { status: "yes", label: "WEB", title: "Acesso: WEB" };
        }
        if (value === "none" || value === "no" || value === "false") {
          return { status: "no", label: "NÃO", title: "Acesso: Nenhum" };
        }
        const label = normalizeLabel(value);
        return { status: "custom", label, title: `Acesso: ${label}` };
      }

      if (raw === true) {
        return { status: "yes", label: "WEB", title: "Acesso: WEB" };
      }

      if (raw === false) {
        return { status: "no", label: "NÃO", title: "Acesso: Nenhum" };
      }

      // fallback: se tiver URL, assumimos WEB
      if (this.item?.url) {
        return { status: "yes", label: "WEB", title: "Acesso: WEB" };
      }

      return { status: "no", label: "NÃO", title: "Acesso: Nenhum" };
    },
    note() {
      return (
        this.item?.note || this.item?.anotacao || this.item?.annotation || ""
      );
    },
    detailsText() {
      const lines = [];
      const name = this.item?.name || "(sem nome)";
      lines.push(`Serviço: ${name}`);

      if (this.item?.subtitle) lines.push(`Subtitle: ${this.item.subtitle}`);
      lines.push(`Acesso: ${this.webPanelInfo.label}`);
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
  beforeUnmount() {
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
        }, 1200);
      };

      try {
        await navigator.clipboard.writeText(text);
        setCopied();
        return;
      } catch {
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
  },
};
</script>

<style scoped lang="scss">
.service-card {
  position: relative;
}

.service-corner-actions {
  position: absolute;
  top: 0.5rem;
  right: 0.5rem;
  z-index: 5;
  display: flex;
  flex-direction: column;
  gap: 0.25rem;
}

.service-action-button {
  width: 24px;
  height: 24px;
  border-radius: 999px;
  border: none;
  background: transparent;
  color: color-mix(in srgb, var(--text-title) 65%, transparent);
  display: inline-flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  padding: 0;

  &:hover {
    color: var(--text-title);
    background: color-mix(in srgb, var(--card-background) 70%, transparent);
  }

  &.is-copy i {
    font-size: 0.85rem;
  }

  &.is-copied {
    color: #2f855a;
  }

  i {
    font-size: 0.95rem;
    line-height: 1;
  }
}

.service-panel-indicator {
  width: 24px;
  height: 24px;
  border-radius: 999px;
  display: inline-flex;
  align-items: center;
  justify-content: center;
  background: color-mix(in srgb, var(--card-background) 70%, transparent);

  i {
    font-size: 0.85rem;
    line-height: 1;
  }

  &.is-yes {
    color: #2f855a;
  }

  &.is-no {
    color: #c53030;
  }

  &.is-custom {
    color: color-mix(in srgb, var(--text-title) 85%, transparent);
    font-size: 0.6rem;
    font-weight: 700;
    letter-spacing: 0.04em;
    text-transform: uppercase;
  }
}

.media-left {
  .image {
    display: flex;
    align-items: center;
  }

  img {
    max-height: 100%;
    object-fit: contain;
  }
}

a[href=""] {
  pointer-events: none;
  cursor: default;
}

.quicklinks {
  float: right;
  a {
    font-size: 0.75rem;
    padding: 3px 6px;
    margin-left: 6px;
    border-radius: 100px;
    background-color: var(--background);
    z-index: 9999;
    pointer-events: all;
  }
}
</style>
