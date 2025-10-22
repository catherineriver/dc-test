<template>
  <div class="modal settings-popup" @click.self="close()">
    <div class="modal__wrapper entire-screen">
      <div class="filters-wrapper">
        <div class="modal__header">
          <div class="title">
            {{ $t("labels.gameSettings") }}
          </div>
          <img
            class="close"
            src="../../assets/lobby/close.svg"
            @click="close"
          />
        </div>
        <div class="settings-content">
          <div class="settings-content-title">
            <img alt="" class="img" src="../../assets/header/language.svg" />
            <span>{{ $t("labels.language") }}</span>
          </div>
          <div class="settings-content-list">
            <LanguageSwitcher />
          </div>
          <div class="settings-content-title">
            <img alt="" class="img" src="../../assets/header/control.svg" />
            <span>{{ $t("labels.control") }}</span>
          </div>
          <div class="settings-content-list">
            <label class="label">
              <div class="label-title">{{ $t("labels.autoThrow") }}</div>
              <input
                v-model="autoThrow"
                name="autoThrow"
                type="checkbox"
                @change="setAutoThrow"
              />
              <div class="box" />
            </label>
            <label id="pieceHighlightSettingCheckbox" class="label">
              <div class="label-title">{{ $t("labels.pieceHighlight") }}</div>
              <input
                v-model="figureHighlight"
                name="figureHighlight"
                type="checkbox"
                @change="setFigureHighlight"
              />
              <div class="box" />
            </label>
            <label id="movementHintsSettingCheckbox" class="label">
              <div class="label-title">{{ $t("labels.movementHints") }}</div>
              <input
                v-model="moveBacklight"
                name="moveBacklight"
                type="checkbox"
                @change="setMoveBacklight"
              />
              <div class="box" />
            </label>
            <label class="label">
              <div class="label-title">
                {{ $t("labels.numbersInsteadOfPieces") }}
              </div>
              <input
                v-model="numbersInsteadFigures"
                name="numbersInsteadFigures"
                type="checkbox"
                @change="setNumbersInsteadFigures"
              />
              <div class="box" />
            </label>
            <label v-if="!isGuest" class="label">
              <div class="label-title">
                {{ $t("labels.twoFactorAuthentication") }}
              </div>
              <input
                v-model="mfa"
                name="mfa"
                type="checkbox"
                @change="setMfa"
              />
              <div class="box" />
            </label>
          </div>
          <div class="settings-content-title">
            <img alt="" class="img" src="../../assets/header/sounds.svg" />
            <span>{{ $t("labels.sound") }}</span>
          </div>
          <div class="settings-content-list">
            <label class="label">
              <span class="label-title">{{ $t("labels.diceRoll") }}</span>
              <input
                v-model="soundDiceRoll"
                name="diceRoll"
                type="checkbox"
                @change="setSoundDiceRoll"
              />
              <div class="box" />
            </label>
            <label class="label">
              <span class="label-title">{{ $t("labels.move") }}</span>
              <input
                v-model="soundMove"
                name="move"
                type="checkbox"
                @change="setSoundMove"
              />
              <div class="box" />
            </label>
            <label class="label">
              <span class="label-title">{{ $t("labels.notifications") }}</span>
              <input
                v-model="soundNotifications"
                name="notifications"
                type="checkbox"
                @change="setSoundNotifications"
              />
              <div class="box" />
            </label>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { Component, Vue } from "vue-property-decorator";
import { Getter } from "vuex-class";
import store, { RootGetter, RootMutation } from "@/store";
import { SecurityRole } from "@/store/SecurityRole";
import AuthorizationService from "@/core/AuthorizationService";
import LoginService from "@/modules/global/LoginService";
import LanguageSwitcher from "@/components/LanguageSwitcher.vue";

@Component({
  components: {
    LanguageSwitcher,
  },
})
export default class SettingsPopup extends Vue {
  private moveBacklight = false;
  private figureHighlight = false;
  private autoThrow = false;
  private soundDiceRoll = false;
  private soundMove = false;
  private soundNotifications = false;
  private numbersInsteadFigures = false;
  private mfa = false;

  @Getter(RootGetter.GET_SETTINGS_AUTO_THROW) settingsAutoThrow!: boolean;
  @Getter(RootGetter.GET_USERNAME) username!: string;
  @Getter(RootGetter.GET_USER_ID) userId!: number;
  @Getter(RootGetter.GET_SETTINGS_MOVE_BACKLIGHT)
  settingsMoveBacklight!: boolean;
  @Getter(RootGetter.GET_SETTINGS_FIGURE_HIGHLIGHT)
  settingsFigureHighlight!: boolean;
  @Getter(RootGetter.GET_SETTINGS_NUMBERS_INSTEAD_FIGURES)
  settingsNumbersInsteadFigures!: boolean;
  @Getter(RootGetter.GET_SETTINGS_SOUND_DICE_ROLL)
  settingsSoundDiceRoll!: boolean;
  @Getter(RootGetter.GET_SETTINGS_SOUND_MOVE) settingsSoundMove!: boolean;
  @Getter(RootGetter.GET_SETTINGS_SOUND_NOTIFICATIONS)
  settingsSoundNotifications!: boolean;
  @Getter(RootGetter.GET_SECURITY_ROLE) securityRole!: SecurityRole;
  @Getter(RootGetter.GET_SETTINGS_MFA) settingsMfa!: boolean;
  @Getter(RootGetter.GET_USER_MFA) userMfa!: boolean;

  async created() {
    this.moveBacklight = this.settingsMoveBacklight;
    this.figureHighlight = this.settingsFigureHighlight;
    this.autoThrow = this.settingsAutoThrow;
    this.soundDiceRoll = this.settingsSoundDiceRoll;
    this.soundMove = this.settingsSoundMove;
    this.soundNotifications = this.settingsSoundNotifications;
    this.numbersInsteadFigures = this.settingsNumbersInsteadFigures;
    this.mfa = this.userMfa;
  }

  private isGuest = LoginService.isGuest();

  public close() {
    this.$store.commit(RootMutation.SET_SETTINGS_IS_VISIBLE, false);
    this.$store.commit(RootMutation.SET_OVERLAY_IS_VISIBLE, false);
  }

  setMoveBacklight() {
    const storage = this.getStorage();
    storage.removeItem("moveBacklight");
    storage.setItem("moveBacklight", String(this.moveBacklight));
    store.commit(RootMutation.SET_SETTINGS_MOVE_BACKLIGHT, this.moveBacklight);
  }

  setFigureHighlight() {
    const storage = this.getStorage();
    storage.removeItem("figureHighlight");
    storage.setItem("figureHighlight", String(this.figureHighlight));
    store.commit(
      RootMutation.SET_SETTINGS_FIGURE_HIGHLIGHT,
      this.figureHighlight
    );
  }

  setAutoThrow() {
    const storage = this.getStorage();
    storage.removeItem("autoThrow");
    storage.setItem("autoThrow", String(this.autoThrow));
    store.commit(RootMutation.SET_SETTINGS_AUTO_THROW, this.autoThrow);
  }

  setSoundDiceRoll() {
    const storage = this.getStorage();
    storage.removeItem("soundDiceRoll");
    storage.setItem("soundDiceRoll", String(this.soundDiceRoll));
    store.commit(RootMutation.SET_SETTINGS_SOUND_DICE_ROLL, this.soundDiceRoll);
  }

  setSoundMove() {
    const storage = this.getStorage();
    storage.removeItem("soundMove");
    storage.setItem("soundMove", String(this.soundMove));
    store.commit(RootMutation.SET_SETTINGS_SOUND_MOVE, this.soundMove);
  }

  setSoundNotifications() {
    const storage = this.getStorage();
    storage.removeItem("soundNotifications");
    storage.setItem("soundNotifications", String(this.soundNotifications));
    store.commit(
      RootMutation.SET_SETTINGS_SOUND_NOTIFICATIONS,
      this.soundNotifications
    );
  }

  setNumbersInsteadFigures() {
    const storage = this.getStorage();
    storage.removeItem("numbersInsteadFigures");
    storage.setItem(
      "numbersInsteadFigures",
      String(this.numbersInsteadFigures)
    );
    store.commit(
      RootMutation.SET_SETTINGS_NUMBERS_INSTEAD_FIGURES,
      this.numbersInsteadFigures
    );
  }

  setMfa() {
    const params = {
      userId: this.userId,
      isMfaEnabled: this.mfa,
    };
    store.commit(RootMutation.SET_USER_DATA, params);
    try {
      AuthorizationService.updateMfaIsEnabled(this.userId, this.mfa);
    } catch (e) {
      console.error(e);
    }
  }

  getStorage() {
    return window.localStorage;
  }
}
</script>
<style lang="scss">
@use 'sass:math';

$scale: 1.9;
$scale-mobile: 2.56;

@function scaled($size) {
  @return math.round($size * $scale);
}

@function scaledMobile($size) {
  @return math.round($size * $scale-mobile);
}

.settings-popup {
  .title {
    font-weight: 600;
    font-size: scaled(24px);
    line-height: scaled(32px);
  }

  .modal__wrapper {
    display: flex;
    flex-direction: column;
    gap: scaled(24px);
    max-width: scaled(603px);
  }

  .modal__header {
    display: flex;
    flex-direction: row;
    align-items: center;
    gap: 30px;
    margin-bottom: 61px;
  }

  .settings-content-title {
    color: #2D2926;
    font-size: scaled(16px);
    font-weight: 600;
    height: scaled(24px);
    background: #EBE8DF;
    display: flex;
    flex-direction: row;
    align-items: center;
    padding: scaled(12px) scaled(6px);

    span {
      vertical-align: middle;
    }
    img {
      width: scaled(27px);
      margin-right: scaled(9px);
      vertical-align: middle;
    }
  }

  .label {
    display: block;
    position: relative;

    input {
      display: none;
    }
    .box {
      border-radius: scaled(6px);
      background: #fff;
      border: scaled(1px) solid #BCC6C2;
      width: scaled(24px);
      height: scaled(24px);
      position: absolute;
      right: scaled(0px);
      top: 50%;
      transform: translateY(-50%);
      transition: .2s;

      &:after {
        content: '';
        width: 100%;
        height: 100%;
        transition: .2s;
        position: absolute;
        left: 0;
        top: 0;
        opacity: 0;
        background-color: #00754D;
        background-image: url("data:image/svg+xml,%3Csvg width='24' height='24' viewBox='0 0 24 24' fill='none' xmlns='http://www.w3.org/2000/svg'%3E%3Cpath d='M9.55 18.15C9.35 18.15 9.15417 18.1125 8.9625 18.0375C8.77083 17.9625 8.6 17.85 8.45 17.7L4.15 13.4C3.85 13.1 3.70417 12.7291 3.7125 12.2875C3.72083 11.8458 3.875 11.475 4.175 11.175C4.475 10.875 4.85 10.725 5.3 10.725C5.75 10.725 6.125 10.875 6.425 11.175L9.55 14.3L17.575 6.27495C17.875 5.97495 18.2458 5.82079 18.6875 5.81245C19.1292 5.80412 19.5083 5.95829 19.825 6.27495C20.125 6.57495 20.275 6.94995 20.275 7.39995C20.275 7.84995 20.125 8.22495 19.825 8.52495L10.65 17.7C10.5 17.85 10.3292 17.9625 10.1375 18.0375C9.94583 18.1125 9.75 18.15 9.55 18.15Z' fill='white'/%3E%3C/svg%3E%0A");
        background-size: scaled(24px);
        background-position: center;
        background-repeat: no-repeat;
        border-radius: scaled(5px);
      }
    }
    input:checked + .box {
      border-color: #00754D;
      background:#00754D;

      &:after {
        opacity: 1;
      }
    }

    input[type="radio"] {
      display: none;
    }
    .circle {
      border-radius: scaled(60px);
      background: #fff;
      border: scaled(1px) solid #BCC6C2;
      width: scaled(24px);
      height: scaled(24px);
      position: absolute;
      right: 0;
      top: 50%;
      transform: translateY(-50%);
      transition: .2s;

      &:after {
        content: '';
        width: 60%;  // Adjust the size to be smaller than the circle itself
        height: 60%; // Same as width to make it a dot
        transition: .2s;
        position: absolute;
        top: 50%;
        left: 50%;
        transform: translate(-50%, -50%); // This will center the dot
        opacity: 0;
        background-color: #006744;
        background-size: scaled(6px);
        background-position: center;
        background-repeat: no-repeat;
        border-radius: scaled(60px);
      }
    }
    input[type="radio"]:checked + .circle {
      border-color: #00754D;
      background: rgba(255, 255, 255, 0);

      &:after {
        opacity: 1;
      }
    }
  }
  .settings-content-list {
    padding: scaled(24px) 0;
    display: flex;
    flex-direction: column;
    gap: scaled(24px);

    .language-switcher {
      display: flex;
      flex-direction: column;
      gap: scaled(24px);
    }
  }
  .language-title {
    color: #2D2926;
    font-family: 'Nunito', serif;
    font-size: scaled(20px);
    line-height: scaled(42px);
    padding-left: 0;
    white-space: nowrap;
  }
  .label-title {
    font-size: scaled(16px);
    line-height: scaled(24px);
  }

}

.vtablet {
  .modal__wrapper {
    max-width: none;
  }

  .label-title, .language-title {
    font-size: scaledMobile(16px);
    line-height: scaledMobile(42px);
  }

  .settings-content-title {
    font-size: scaledMobile(16px);
    height: scaledMobile(40px);
    padding: 0 scaledMobile(16px);

    img {
      width: scaledMobile(24px);
    }
  }
}

.phone {
  .settings-popup {


    .modal__header {
      padding: scaledMobile(16px) scaledMobile(16px);
      margin-bottom: 0;
    }

    .modal__wrapper {
      padding: 0;
      max-width: none;
      overflow: hidden;
    }
    .create-game__item {
      padding: scaledMobile(16px) scaledMobile(16px) 0 scaledMobile(16px);
    }
    .settings-content-list {
      display: flex;
      flex-direction: column;
      gap: scaledMobile(0px);
      margin: scaledMobile(24px) 0;

      padding: 0 scaledMobile(16px)
    }

    .settings-content-title {
      font-size: scaledMobile(16px);
      height: scaledMobile(40px);
      padding: 0 scaledMobile(16px);

      img {
        width: scaledMobile(24px);
      }
    }

    .settings-content {
      max-height: scaledMobile(740px);
      overflow-y: scroll;
    }

    .label-title, .language-title {
      font-size: scaledMobile(16px);
      line-height: scaledMobile(42px);
    }

    .box {
      border-radius: scaledMobile(6px);
      border: scaledMobile(1px) solid #BCC6C2;
      width: scaledMobile(24px);
      height: scaledMobile(24px);
      right: scaledMobile(0px);

      &:after {
        background-size: scaledMobile(24px);
        border-radius: scaledMobile(5px);
      }
    }

    .circle {
      border-radius: scaledMobile(60px);
      border: scaledMobile(1px) solid #BCC6C2;
      width: scaledMobile(24px);
      height: scaledMobile(24px);

      &:after {
        background-size: scaledMobile(6px);
        border-radius: scaledMobile(60px);
      }
    }
  }
}


</style>
