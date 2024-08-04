<template>
<div class="toolbar">
  <div class="mt-4">
    <b-button-group>
      <b-button
        v-if="$store.state.ffmpegdEnabled && wsReady"
        class="ml-2 float-right"
        variant="outline-primary"
        @click="encode">{{ encoding ? 'Encoding...' : 'Encode' }}
      </b-button>

      <b-dropdown
        variant="outline-primary"
        split
        :text="copied ? '已複製' : '複製'"
        v-b-tooltip.hover.bottom title="複製到剪貼簿"
        @click="copyToClipboard">
        <b-dropdown-item @click="toggleJSON">
          {{ value.showJSON ? '隱藏' : '顯示' }} JSON
        </b-dropdown-item>
      </b-dropdown>
    </b-button-group>

    <b-button-group class="float-right">
      <b-button
        class="ml-2 float-right"
        variant="outline-danger"
        @click="$emit('reset')">重置
      </b-button>

      <b-dropdown
        variant="outline-primary"
        split
        v-b-tooltip.hover.bottomright title="儲存至 Local Storage"
        :text="saving ? '儲存中...' : '儲存'" @click="save"
      >
        <b-dropdown-item @click="$emit('save', true)">
          儲存成新的
        </b-dropdown-item>
        <b-dropdown-item v-if="preset.name" @click="deletePreset">
          刪除設定擋
        </b-dropdown-item>
        <b-dropdown-divider></b-dropdown-divider>
        <b-dropdown-item @click="deleteAllPresetsPrompt">
          刪除所有已儲存設定擋
        </b-dropdown-item>
      </b-dropdown>
    </b-button-group>
  </div>

  <!-- Overlay prompt when deleting all saved presets. -->
  <b-overlay :show="showDeletePrompt" no-wrap @shown="onShown">
    <template v-slot:overlay>
    <div ref="dialog" class="text-center p-3">
      <p>您確定要刪除所有設定擋?</p>
      <div>
        <b-button class="mr-3" @click="onCancel">Cancel</b-button>
        <b-button variant="outline-danger" @click="onOK">刪除所有設定擋</b-button>
      </div>
    </div>
    </template>
  </b-overlay>
</div>
</template>

<script>
import presets from '@/presets';

export default {
  name: 'Toolbar',
  props: ['value', 'preset'],
  computed: {
    wsReady() {
      return this.$store.state.wsConnected;
    },
  },
  data() {
    return {
      encoding: false,
      copied: false,
      saving: false,
      showDeletePrompt: false,
    };
  },
  methods: {
    encode() {
      this.encoding = true;
      setTimeout(() => {
        this.encoding = false;
        this.$emit('encode');
      }, 500);
    },
    copyToClipboard() {
      const copyText = this.$parent.$refs.code;
      copyText.select();
      document.execCommand('copy');

      // Update copy button text.
      this.copied = true;
      setTimeout(() => {
        this.copied = false;
      }, 1000);
    },
    toggleJSON() {
      this.$emit('toggleJSON');
    },
    deletePreset() {
      presets.deletePreset(this.preset.id);
      presets.getPresetOptions();
      this.$emit('reset');
    },
    deleteAllPresetsPrompt() {
      this.showDeletePrompt = true;
    },
    deleteAllPresets() {
      presets.deleteAllPresets();
      this.$emit('reset');
    },
    save() {
      this.saving = true;
      this.$emit('save', false);
      setTimeout(() => {
        this.saving = false;
      }, 1000);
    },
    onShown() {
      this.$refs.dialog.focus();
    },
    onCancel() {
      this.showDeletePrompt = false;
    },
    onOK() {
      this.deleteAllPresets();
      presets.getPresetOptions();
      this.showDeletePrompt = false;
    },
  },
};
</script>
