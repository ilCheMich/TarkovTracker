<template>
  <KeepAlive>
    <v-lazy :options="{ threshold: 0.5 }" min-height="64">
      <v-sheet rounded :class="itemRowClasses">
        <v-container fluid class="my-n1 pa-0">
          <v-row no-gutters>
            <v-col class="d-flex align-center pa-0" cols="9" xs="9" sm="9" md="6" lg="6" xl="6"
              style="overflow:-moz-hidden-unscrollable">
              <span class="d-block">
                <v-img :src="item.iconLink" :class="itemImageClasses">
                  <template #placeholder>
                    <v-row class="fill-height ma-0" align="center" justify="center">
                      <v-progress-circular indeterminate color="grey-lighten-5"></v-progress-circular>
                    </v-row>
                  </template>
                </v-img>
              </span>
              <span class="ml-2 d-flex flex-column"
                style="-webkit-mask-image: linear-gradient(90deg, #000 100%, transparent);">
                <span style="font-size: 1.1em">
                  {{ item.name }}
                </span>
                <span>
                  <template v-if="props.need.needType == 'taskObjective'">
                    <TaskLink :task="relatedTask" />
                  </template>
                  <template v-else-if="props.need.needType == 'hideoutModule'">
                    <StationLink :station="relatedStation" />
                  </template>
                </span>
              </span>
            </v-col>
            <v-col cols="3" xs="3" sm="3" md="6" lg="6" xl="6" class="d-flex flex-column align-end justify-center">
              <div v-if="smAndDown" class="d-block mr-2">
                <v-btn variant="tonal" class="pa-0 px-1 ma-0">
                  {{ currentCount.toLocaleString() }}/{{ neededCount.toLocaleString() }}
                  <v-dialog v-model="smallDialog" activator="parent" width="80%" scrim="#9A8866">
                    <v-sheet>
                      <div class="d-flex align-end flex-column fill-height">
                        <!-- Item image -->
                        <div class="d-flex align-self-stretch item-panel">
                          <v-img :src="item.image512pxLink" :lazy-src="item.baseImageLink"
                            :class="itemImageDialogClasses">
                            <template #placeholder>
                              <v-row class="fill-height ma-0" align="center" justify="center">
                                <v-progress-circular indeterminate color="grey-lighten-5"></v-progress-circular>
                              </v-row>
                            </template>
                          </v-img>
                        </div>
                        <div class="d-flex align-self-center mt-2 mx-2">
                          <div class="text-center px-2">
                            {{ item.name }}
                          </div>
                        </div>
                        <!-- Item need details -->
                        <div class="d-flex flex-column align-self-center mt-2 mx-2">
                          <template v-if="props.need.needType == 'taskObjective'">
                            <task-link :task="relatedTask" />
                            <v-row v-if="relatedTask.predecessors?.length > 0" no-gutters
                              class="mb-1 mt-1 d-flex justify-center">
                              <v-col cols="auto" class="mr-1" align="center">
                                <v-icon icon="mdi-lock-open-outline" />
                              </v-col>
                              <v-col cols="auto" align="center">
                                <i18n-t keypath="page.tasks.questcard.lockedbefore" scope="global">
                                  <template #count>
                                    {{ lockedBefore }}
                                  </template>
                                </i18n-t>
                              </v-col>
                            </v-row>
                            <v-row v-if="levelRequired > 0 && levelRequired > tarkovStore.playerLevel" no-gutters
                              class="mb-1 mt-1 d-flex justify-center">
                              <v-col cols="auto" class="mr-1" align="center">
                                <v-icon icon="mdi-menu-right" />
                              </v-col>
                              <v-col cols="auto" align="center">
                                <i18n-t keypath="page.tasks.questcard.level" scope="global">
                                  <template #count>
                                    {{ levelRequired }}
                                  </template>
                                </i18n-t>
                              </v-col>
                            </v-row>
                          </template>
                          <template v-else-if="props.need.needType == 'hideoutModule'">
                            <v-row dense no-gutters class="mb-1 mt-1 d-flex justify-center">
                              <v-col cols="auto" align="center">
                                <station-link :station="relatedStation" class="justify-center" />
                              </v-col>
                              <v-col cols="auto" class="ml-1">{{
                                props.need.hideoutModule.level
                              }}</v-col>
                            </v-row>
                            <v-row v-if="props.need.hideoutModule.predecessors?.length > 0" no-gutters
                              class="mb-1 mt-1 d-flex justify-center">
                              <v-col cols="auto" class="mr-1" align="center">
                                <v-icon icon="mdi-lock-open-outline" />
                              </v-col>
                              <v-col cols="auto" align="center">
                                <i18n-t keypath="page.tasks.questcard.lockedbefore" scope="global">
                                  <template #count>
                                    {{ lockedBefore }}
                                  </template>
                                </i18n-t>
                              </v-col>
                            </v-row>
                            <v-row v-if="levelRequired > 0 && levelRequired > tarkovStore.playerLevel" no-gutters
                              class="mb-1 mt-1 d-flex justify-center">
                              <v-col cols="auto" class="mr-1" align="center">
                                <v-icon icon="mdi-menu-right" />
                              </v-col>
                              <v-col cols="auto" align="center">
                                <i18n-t keypath="page.tasks.questcard.level" scope="global">
                                  <template #count>
                                    {{ levelRequired }}
                                  </template>
                                </i18n-t>
                              </v-col>
                            </v-row>
                          </template>
                        </div>
                        <!-- Item count actions -->
                        <div class="d-flex align-self-stretch justify-center mt-auto mb-2 mx-2">
                          <div>
                            <v-btn variant="tonal" class="pa-0 ma-0"
                              @click="decreaseCount()"><v-icon>mdi-minus-thick</v-icon></v-btn>
                          </div>
                          <div class="mx-1">
                            <v-btn variant="tonal" class="pa-0 px-1 ma-0" @click="toggleCount()">
                              {{ currentCount.toLocaleString() }}/{{ neededCount.toLocaleString() }}
                            </v-btn>
                          </div>
                          <div>
                            <v-btn variant="tonal" class="pa-0 ma-0"
                              @click="increaseCount()"><v-icon>mdi-plus-thick</v-icon></v-btn>
                          </div>
                        </div>
                      </div>
                    </v-sheet>
                  </v-dialog>
                </v-btn>
              </div>
              <div v-else class="d-flex flex-row">
                <div v-if="mdAndUp" class="d-flex align-self-center justify-space-between mr-2">
                  <template v-if="props.need.needType == 'taskObjective'">
                    <div v-if="levelRequired > 0 && levelRequired > tarkovStore.playerLevel"
                      class="d-flex align-center mb-1 mt-1 mr-2 d-flex justify-center">
                      <div>
                        <v-icon icon="mdi-menu-right" />
                      </div>
                      <div>
                        <i18n-t keypath="page.tasks.questcard.level" scope="global">
                          <template #count>
                            {{ levelRequired }}
                          </template>
                        </i18n-t>
                      </div>
                    </div>
                    <div v-if="relatedTask.predecessors?.length > 0" no-gutters
                      class="mb-1 mt-1 d-flex align-center justify-center">
                      <div>
                        <v-icon icon="mdi-lock-open-outline" />
                      </div>
                      <div>
                        <i18n-t keypath="page.tasks.questcard.lockedbefore" scope="global">
                          <template #count>
                            {{ lockedBefore }}
                          </template>
                        </i18n-t>
                      </div>
                    </div>
                  </template>
                  <template v-else-if="props.need.needType == 'hideoutModule'">
                    <div class="d-flex align-center mr-2">
                      <v-row dense no-gutters class="mb-1 mt-1 d-flex justify-center">
                        <v-col cols="auto" align="center">
                          <station-link :station="relatedStation" class="justify-center" />
                        </v-col>
                        <v-col cols="auto" class="ml-1">{{
                          props.need.hideoutModule.level
                        }}</v-col>
                      </v-row>
                    </div>
                    <div v-if="props.need.hideoutModule.predecessors?.length > 0" no-gutters
                      class="mb-1 mt-1 d-flex justify-center">
                      <div>
                        <v-icon icon="mdi-lock-open-outline" />
                      </div>
                      <div>
                        <i18n-t keypath="page.tasks.questcard.lockedbefore" scope="global">
                          <template #count>
                            {{ lockedBefore }}
                          </template>
                        </i18n-t>
                      </div>
                    </div>
                    <div v-if="levelRequired > 0" no-gutters class="mb-1 mt-1 d-flex justify-center">
                      <div>
                        <v-icon icon="mdi-menu-right" />
                      </div>
                      <div>
                        <i18n-t keypath="page.tasks.questcard.level" scope="global">
                          <template #count>
                            {{ levelRequired }}
                          </template>
                        </i18n-t>
                      </div>
                    </div>
                  </template>
                </div>
                <div class="d-flex align-self-center justify-space-between mr-2">
                  <div>
                    <v-btn variant="tonal" class="pa-0 ma-0"
                      @click="decreaseCount()"><v-icon>mdi-minus-thick</v-icon></v-btn>
                  </div>
                  <div class="mx-1">
                    <v-btn variant="tonal" class="pa-0 px-1 ma-0" @click="toggleCount()">
                      {{ currentCount.toLocaleString() }}/{{ neededCount.toLocaleString() }}
                    </v-btn>
                  </div>
                  <div>
                    <v-btn variant="tonal" class="pa-0 ma-0"
                      @click="increaseCount()"><v-icon>mdi-plus-thick</v-icon></v-btn>
                  </div>
                </div>
              </div>
            </v-col>
          </v-row>
        </v-container>
      </v-sheet>
    </v-lazy>
  </KeepAlive>
</template>
<script setup>
import { defineAsyncComponent, computed, inject, ref } from "vue";
import { useUserStore } from "@/stores/user";
import { useProgressStore } from "@/stores/progress";
import { useTarkovData } from "@/composables/tarkovdata";
import { useTarkovStore } from "@/stores/tarkov";
import { useDisplay } from "vuetify";
const TaskLink = defineAsyncComponent(() =>
  import("@/components/tasks/TaskLink.vue")
);
const StationLink = defineAsyncComponent(() =>
  import("@/components/hideout/StationLink.vue")
);
const props = defineProps({
  need: {
    type: Object,
    required: true,
  },
});

const { smAndDown, mdAndUp } = useDisplay();

const userStore = useUserStore();
const progressStore = useProgressStore();
const tarkovStore = useTarkovStore();

const { tasks, hideoutStations } = useTarkovData();

const smallDialog = ref(false);

const selfCompletedNeed = computed(() => {
  if (props.need.needType == "taskObjective") {
    return (
      progressStore.tasksCompletions[props.need.taskId]["self"] ||
      progressStore.objectiveCompletions[props.need.id]["self"]
    );
  } else if (props.need.needType == "hideoutModule") {
    return (
      progressStore.moduleCompletions[props.need.hideoutModule.id]["self"] ||
      progressStore.modulePartCompletions[props.need.id]["self"]
    );
  } else {
    return false;
  }
});

const relatedTask = computed(() => {
  if (props.need.needType == "taskObjective") {
    return tasks.value.find((t) => t.id == props.need.taskId);
  } else {
    return null;
  }
});

const relatedStation = computed(() => {
  if (props.need.needType == "hideoutModule") {
    return Object.values(hideoutStations.value).find(
      (s) => s.id == props.need.hideoutModule.stationId
    );
  } else {
    return null;
  }
});

const lockedBefore = computed(() => {
  if (props.need.needType == "taskObjective") {
    return relatedTask.value.predecessors.filter(
      (s) => !tarkovStore.isTaskComplete(s.id)
    ).length;
  } else if (props.need.needType == "hideoutModule") {
    return props.need.hideoutModule.predecessors.filter(
      (s) => !tarkovStore.isHideoutModuleComplete(s.id)
    ).length;
  } else {
    return 0;
  }
});

const itemImageClasses = computed(() => {
  return {
    [`item-bg-${item.value.backgroundColor}`]: true,
    rounded: true,
    "pa-1": true,
    "item-row-image": true,
  };
});

const itemImageDialogClasses = computed(() => {
  return {
    [`item-bg-${item.value.backgroundColor}`]: true,
    rounded: true,
    "pa-1": true,
  };
});

const itemRowClasses = computed(() => {
  return {
    "item-complete": selfCompletedNeed.value || currentCount.value >= neededCount.value,
  };
});

const currentCount = computed(() => {
  if (props.need.needType == "taskObjective") {
    return tarkovStore.getObjectiveCount(props.need.id);
  } else if (props.need.needType == "hideoutModule") {
    return tarkovStore.getHideoutPartCount(props.need.id);
  } else {
    return 0;
  }
});

const decreaseCount = () => {
  if (props.need.needType == "taskObjective") {
    if (currentCount.value > 0) {
      tarkovStore.setObjectiveCount(props.need.id, currentCount.value - 1);
    }
  } else if (props.need.needType == "hideoutModule") {
    if (currentCount.value > 0) {
      tarkovStore.setHideoutPartCount(props.need.id, currentCount.value - 1);
    }
  }
};

const increaseCount = () => {
  if (props.need.needType == "taskObjective") {
    if (currentCount.value < props.need.count) {
      tarkovStore.setObjectiveCount(props.need.id, currentCount.value + 1);
    }
  } else if (props.need.needType == "hideoutModule") {
    if (currentCount.value < props.need.count) {
      tarkovStore.setHideoutPartCount(props.need.id, currentCount.value + 1);
    }
  }
};

const levelRequired = computed(() => {
  if (props.need.needType == "taskObjective") {
    return relatedTask.value.minPlayerLevel;
  } else if (props.need.needType == "hideoutModule") {
    return 0;
  } else {
    return 0;
  }
});

const toggleCount = () => {
  if (props.need.needType == "taskObjective") {
    if (currentCount.value === 0) {
      tarkovStore.setObjectiveCount(props.need.id, props.need.count);
    } else if (currentCount.value === props.need.count) {
      tarkovStore.setObjectiveCount(props.need.id, 0);
    } else {
      tarkovStore.setObjectiveCount(props.need.id, props.need.count);
    }
  } else if (props.need.needType == "hideoutModule") {
    if (currentCount.value === 0) {
      tarkovStore.setHideoutPartCount(props.need.id, props.need.count);
    } else if (currentCount.value === props.need.count) {
      tarkovStore.setHideoutPartCount(props.need.id, 0);
    } else {
      tarkovStore.setHideoutPartCount(props.need.id, props.need.count);
    }
  }
};

const neededCount = computed(() => {
  if (props.need.needType == "taskObjective" && props.need.count) {
    return props.need.count;
  } else if (props.need.needType == "hideoutModule" && props.need.count) {
    return props.need.count;
  } else {
    return 1;
  }
});

const item = computed(() => {
  if (props.need.needType == "taskObjective") {
    if (props.need.type == "mark") {
      return props.need.markerItem;
    } else if (props.need.type == "buildWeapon") {
      return props.need.item;
    } else if (props.need.type == "plantItem") {
      return props.need.item;
    } else if (props.need.type == "giveItem") {
      return props.need.item;
    } else {
      return null;
    }
  } else if (props.need.needType == "hideoutModule") {
    return props.need.item;
  } else {
    return null;
  }
});
</script>
<style lang="scss">
.item-complete {
  background: linear-gradient(270deg,
      rgba(var(--v-theme-complete), 1) 0%,
      rgba(var(--v-theme-surface), 1) 100%) !important;
}

.item-panel {
  aspect-ratio: 16/9;
  min-height: 100px;
}

.item-row-image {
  aspect-ratio: 1/1;
  min-height: 64px;
  max-height: 64px;
}

.item-bg-violet {
  background-color: #2c232f;
}

.item-bg-grey {
  background-color: #1e1e1e;
}

.item-bg-yellow {
  background-color: #343421;
}

.item-bg-orange {
  background-color: #261d14;
}

.item-bg-green {
  background-color: #1a2314;
}

.item-bg-red {
  background-color: #38221f;
}

.item-bg-default {
  background-color: #3a3c3b;
}

.item-bg-black {
  background-color: #141614;
}

.item-bg-blue {
  background-color: #202d32;
}
</style>