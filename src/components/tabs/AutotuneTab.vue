<template>
    <BaseTab tab-name="autotune">
        <div class="content_wrapper grid-box col1">
            <!-- Title -->
            <div class="cf_column">
                <div class="tab_title" v-html="$t('tabAutotune')"></div>
            </div>

            <!-- Overview / How to Use -->
            <UiBox :title="$t('autotuneOverviewTitle')">
                <div class="autotune-overview">
                    <p v-html="$t('autotuneOverviewText')"></p>
                    <div class="autotune-steps">
                        <h4 v-html="$t('autotuneQuickStartTitle')"></h4>
                        <ol>
                            <li v-html="$t('autotuneStep1')"></li>
                            <li v-html="$t('autotuneStep2')"></li>
                            <li v-html="$t('autotuneStep3')"></li>
                            <li v-html="$t('autotuneStep4')"></li>
                            <li v-html="$t('autotuneStep5')"></li>
                        </ol>
                    </div>
                </div>
            </UiBox>

            <!-- Autotune Settings -->
            <UiBox :title="$t('autotuneSettingsTitle')">
                <div class="autotune-settings">
                    <!-- Gain Ramp Rate -->
                    <SettingRow :label="$t('autotuneGainRampRate')" :help="$t('autotuneGainRampRateHelp')">
                        <UInputNumber
                            v-model="advancedTuning.autotuneGainRampRate"
                            :step="1"
                            :min="101"
                            :max="150"
                        />
                    </SettingRow>

                    <!-- Gain Margin -->
                    <SettingRow :label="$t('autotuneGainMargin')" :help="$t('autotuneGainMarginHelp')">
                        <UInputNumber
                            v-model="advancedTuning.autotuneGainMargin"
                            :step="1"
                            :min="30"
                            :max="90"
                        />
                    </SettingRow>

                    <!-- Oscillation Threshold -->
                    <SettingRow :label="$t('autotuneOscThreshold')" :help="$t('autotuneOscThresholdHelp')">
                        <UInputNumber
                            v-model="advancedTuning.autotuneOscThreshold"
                            :step="1"
                            :min="5"
                            :max="100"
                        />
                    </SettingRow>

                    <!-- P:I Ratio -->
                    <SettingRow :label="$t('autotunePiRatio')" :help="$t('autotunePiRatioHelp')">
                        <UInputNumber
                            v-model="advancedTuning.autotunePiRatio"
                            :step="1"
                            :min="20"
                            :max="200"
                        />
                    </SettingRow>

                    <!-- Max Gain Multiplier -->
                    <SettingRow :label="$t('autotuneMaxGainMult')" :help="$t('autotuneMaxGainMultHelp')">
                        <UInputNumber
                            v-model="advancedTuning.autotuneMaxGainMultiplier"
                            :step="1"
                            :min="10"
                            :max="100"
                        />
                    </SettingRow>

                    <!-- Settle Time -->
                    <SettingRow :label="$t('autotuneSettleTime')" :help="$t('autotuneSettleTimeHelp')">
                        <UInputNumber
                            v-model="advancedTuning.autotuneSettleTimeMs"
                            :step="50"
                            :min="100"
                            :max="5000"
                        />
                    </SettingRow>

                    <!-- Timeout -->
                    <SettingRow :label="$t('autotuneTimeout')" :help="$t('autotuneTimeoutHelp')">
                        <UInputNumber
                            v-model="advancedTuning.autotuneTimeoutMs"
                            :step="1000"
                            :min="5000"
                            :max="60000"
                        />
                    </SettingRow>

                    <!-- Tune Yaw -->
                    <SettingRow :label="$t('autotuneTuneYaw')" :help="$t('autotuneTuneYawHelp')">
                        <div class="autotune-toggle">
                            <select v-model.number="advancedTuning.autotuneTuneYaw">
                                <option :value="0" v-html="$t('off')"></option>
                                <option :value="1" v-html="$t('on')"></option>
                            </select>
                        </div>
                    </SettingRow>
                </div>
            </UiBox>

            <!-- Tips & Safety -->
            <UiBox :title="$t('autotuneTipsTitle')">
                <div class="autotune-tips">
                    <ul>
                        <li v-html="$t('autotuneTip1')"></li>
                        <li v-html="$t('autotuneTip2')"></li>
                        <li v-html="$t('autotuneTip3')"></li>
                        <li v-html="$t('autotuneTip4')"></li>
                        <li v-html="$t('autotuneTip5')"></li>
                    </ul>
                </div>
            </UiBox>
        </div>

        <!-- Bottom Toolbar -->
        <div class="content_toolbar toolbar_fixed_bottom">
            <div class="btn save_btn">
                <button type="button" class="save" @click="save" v-html="$t('autotuneButtonSave')"></button>
            </div>
        </div>
    </BaseTab>
</template>

<script>
import { defineComponent, onMounted, computed, nextTick } from "vue";
import BaseTab from "./BaseTab.vue";
import UiBox from "../elements/UiBox.vue";
import SettingRow from "../elements/SettingRow.vue";
import GUI from "../../js/gui";
import FC from "../../js/fc";
import MSP from "../../js/msp";
import MSPCodes from "../../js/msp/MSPCodes";
import { mspHelper } from "../../js/msp/MSPHelper";
import { i18n } from "../../js/localization";
import { gui_log } from "../../js/gui_log";

export default defineComponent({
    name: "AutotuneTab",
    components: {
        BaseTab,
        UiBox,
        SettingRow,
    },
    setup() {
        const advancedTuning = computed(() => FC.ADVANCED_TUNING);

        async function loadData() {
            try {
                await MSP.promise(MSPCodes.MSP_PID_ADVANCED);
                nextTick(() => {
                    i18n.localizePage();
                    GUI.content_ready();
                });
            } catch (e) {
                console.error("[Autotune] Load failed:", e);
            }
        }

        async function save() {
            try {
                await MSP.promise(
                    MSPCodes.MSP_SET_PID_ADVANCED,
                    mspHelper.crunch(MSPCodes.MSP_SET_PID_ADVANCED),
                );
                await MSP.promise(MSPCodes.MSP_EEPROM_WRITE);
                gui_log(i18n.getMessage("autotuneSettingsSaved"));
                await loadData();
            } catch (e) {
                console.error("[Autotune] Save failed:", e);
            }
        }

        onMounted(async () => {
            await loadData();
        });

        return {
            advancedTuning,
            save,
        };
    },
});
</script>

<style scoped>
.autotune-overview {
    padding: 8px 0;
    line-height: 1.6;
}

.autotune-overview p {
    margin-bottom: 12px;
    color: var(--subtleAccent);
}

.autotune-steps {
    margin-top: 8px;
}

.autotune-steps h4 {
    margin-bottom: 6px;
    font-weight: 600;
}

.autotune-steps ol {
    margin-left: 20px;
    padding-left: 0;
}

.autotune-steps ol li {
    margin-bottom: 4px;
    line-height: 1.5;
}

.autotune-settings {
    display: flex;
    flex-direction: column;
    gap: 12px;
    padding: 8px 0;
}

.autotune-toggle select {
    min-width: 80px;
}

.autotune-tips {
    padding: 8px 0;
}

.autotune-tips ul {
    margin-left: 20px;
    padding-left: 0;
}

.autotune-tips ul li {
    margin-bottom: 6px;
    line-height: 1.5;
    color: var(--subtleAccent);
}
</style>
