<template>
  <div
    class="mb-4 flex flex-col flex-1 flex-grow-0 max-w-full transition-all header"
  >
    <div
      class="flex w-full align-center max-h-20 flex-grow-0 flex-1 items-baseline pl-6 pr-6"
    >
      <span
        :style="{
          fontSize: headerFontSize
        }"
        class="text-shadow-lg transition-all whitespace-nowrap overflow-hidden overflow-ellipsis"
      >{{ name || `Minecraft ${version.minecraft}` }}</span>
      <AvatarItem
        v-ripple
        :color="!localVersion.id ? 'warning' : 'primary'"
        icon="fact_check"
        class="cursor-pointer ml-2"
        :title="t('version.name', 2)"
        :text="currentVersion"
        @click="onShowLocalVersion"
      />
      <div class="flex-grow" />
      <v-tooltip
        :close-delay="0"
        top
        transition="scroll-y-reverse-transition"
      >
        <template #activator="{ on }">
          <v-btn
            text
            icon
            :loading="refreshing"
            v-on="on"
            @click="showExport"
          >
            <v-icon>
              share
            </v-icon>
          </v-btn>
        </template>
        {{ t('modpack.export') }}
      </v-tooltip>

      <v-tooltip
        top
        transition="scroll-y-reverse-transition"
      >
        <template #activator="{ on }">
          <v-btn
            class="ml-1.5"
            text
            icon
            v-on="on"
            @click="showLogDialog"
          >
            <v-icon>
              subtitles
            </v-icon>
          </v-btn>
        </template>
        {{ t("logsCrashes.title") }}
      </v-tooltip>

      <v-tooltip
        top
        transition="scroll-y-reverse-transition"
      >
        <template #activator="{ on }">
          <v-btn
            class="ml-1.5"
            text
            icon
            v-on="on"
            @click="showInstanceFolder"
          >
            <v-icon>
              folder
            </v-icon>
          </v-btn>
        </template>
        {{ t("instance.showInstance") }}
      </v-tooltip>

      <v-tooltip
        top
        transition="scroll-y-reverse-transition"
      >
        <template #activator="{ on }">
          <v-btn
            class="ml-1.5"
            text
            icon
            to="/base-setting"
            v-on="on"
          >
            <v-icon>
              settings
            </v-icon>
          </v-btn>
        </template>
        {{ t('baseSetting.title', 2) }}
      </v-tooltip>
    </div>
    <div
      class="flex flex-grow-0 flex-1 mt-4 flex-row  items-center justify-center pl-4 pr-6"
    >
      <div
        class="flex flex-row items-center gap-1 flex-grow-0 justify-center"
      >
        <AvatarItem
          :avatar="'image://builtin/minecraft'"
          title="Minecraft"
          :text="`${version.minecraft}`"
        />
        <v-divider vertical />
        <AvatarItem
          v-if="version.forge"
          :avatar="'image://builtin/forge'"
          title="Forge"
          :text="`${version.forge}`"
        />
        <v-divider
          v-if="version.forge"
          vertical
        />
        <AvatarItem
          v-if="version.fabricLoader"
          :avatar="'image://builtin/fabric'"
          title="Fabric"
          :text="`${version.fabricLoader}`"
        />
        <v-divider
          v-if="version.fabricLoader"
          vertical
        />
        <AvatarItem
          v-if="version.quiltLoader"
          :avatar="'image://builtin/quilt'"
          title="Quilt"
          :text="`${version.quiltLoader}`"
        />
        <v-divider
          v-if="version.quiltLoader"
          vertical
        />
        <AvatarItem
          v-if="version.optifine"
          :avatar="'image://builtin/optifine'"
          title="Optifine"
          :text="`${version.optifine}`"
        />
        <v-divider
          v-if="version.optifine"
          vertical
        />
        <AvatarItem
          icon="schedule"
          title="游戏时间"
          text="100 小时"
        />
        <v-divider
          vertical
        />
        <AvatarItem
          icon="update"
          title="最后运行"
          :text="lastPlayedText"
        />
      </div>
      <div class="flex-grow" />

      <div
        v-if="!isInFocusMode"
        class="flex align-end gap-7 flex-1 flex-grow-0"
      >
        <HomeHeaderInstallStatus
          v-if="status === 1 || status === 3"
          :name="taskName"
          :total="total"
          :progress="progress"
        />
        <HomeLaunchButton
          :issue="issue"
          :compact="compact"
          :status="status"
          @pause="pause"
          @resume="resume"
        />
      </div>
    </div>

    <v-divider
      class="mt-2 transition-all"
      :class="{
        'mx-4': !compact,
      }"
    />
  </div>
</template>

<script lang=ts setup>
import AvatarItem from '@/components/AvatarItem.vue'
import { useService } from '@/composables'
import { kInstanceContext } from '@/composables/instanceContext'
import { kCompact } from '@/composables/scrollTop'
import { useInFocusMode } from '@/composables/uiLayout'
import { getLocalDateString } from '@/util/date'
import { injection } from '@/util/inject'
import { BaseServiceKey, VersionServiceKey } from '@xmcl/runtime-api'
import { useDialog } from '../composables/dialog'
import { AppExportDialogKey } from '../composables/instanceExport'
import HomeHeaderInstallStatus from './HomeHeaderInstallStatus.vue'
import HomeLaunchButton from './HomeLaunchButton.vue'

const { issue, task, path, refreshing, name, version, localVersion, instance } = injection(kInstanceContext)
const isInFocusMode = useInFocusMode()
const { total, progress, pause, resume, status, name: taskName } = task
const { openDirectory } = useService(BaseServiceKey)
const { show: showLogDialog } = useDialog('log')
const { show: showExport } = useDialog(AppExportDialogKey)
const { t } = useI18n()
const { showVersionDirectory } = useService(VersionServiceKey)

const currentVersion = computed(() => !localVersion.value.id ? t('version.notInstalled') : localVersion.value.id)
const scrollTop = injection(kCompact)
const compact = computed(() => scrollTop.value)
const headerFontSize = computed(() => {
  if (compact.value) {
    return '1.8rem'
  }
  if (name.value && name.value.length > 30) {
    return '2rem'
  }
  return '2.425rem'
})
const lastPlayedText = computed(() => {
  const i = instance.value
  const date = i.lastAccessDate
  return getLocalDateString(date)
})

const onShowLocalVersion = () => {
  if (localVersion.value.id) {
    showVersionDirectory(localVersion.value.id)
  }
}

function showInstanceFolder() {
  openDirectory(path.value)
}

</script>
<style scoped>
.compact {
  background: rgba(0, 0, 0, 0.5);
}

</style>
