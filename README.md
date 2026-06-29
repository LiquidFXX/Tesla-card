type: vertical-stack
cards:
  - type: vertical-stack
    cards:
      - type: custom:button-card
        entity: sensor.coaster_battery_level
        show_name: false
        show_icon: false
        show_state: false
        tap_action:
          action: more-info
        hold_action:
          action: more-info
          entity: lock.coaster_lock
        triggers_update:
          - sensor.coaster_battery_level
          - sensor.coaster_battery_range
          - sensor.coaster_charging
          - sensor.coaster_charger_power_3
          - sensor.coaster_charge_rate
          - sensor.coaster_speed
          - sensor.coaster_power
          - sensor.coaster_shift_state_2
          - number.coaster_charge_limit_2
          - binary_sensor.coaster_status
          - binary_sensor.coaster_charge_cable
          - binary_sensor.coaster_trip_charging
          - binary_sensor.coaster_scheduled_charging_pending
          - binary_sensor.coaster_battery_heater
          - binary_sensor.coaster_preconditioning_enabled
          - binary_sensor.coaster_user_present_2
          - binary_sensor.coaster_dashcam
          - lock.coaster_lock
          - device_tracker.coaster_location
          - climate.coaster_climate
          - switch.coaster_sentry_mode_2
        styles:
          card:
            - min-height: 365px
            - padding: 18px
            - border-radius: 28px
            - overflow: hidden
            - position: relative
            - background: |
                radial-gradient(
                  circle at 61% 34%,
                  rgba(42,235,104,0.16),
                  transparent 30%
                ),
                radial-gradient(
                  circle at 10% 0%,
                  rgba(55,139,255,0.14),
                  transparent 32%
                ),
                linear-gradient(
                  145deg,
                  rgba(8,18,29,0.98),
                  rgba(3,9,16,0.99)
                )
            - border: 1px solid rgba(255,255,255,0.11)
            - box-shadow: |
                0 18px 38px rgba(0,0,0,0.34),
                inset 0 1px 0 rgba(255,255,255,0.06)
          grid:
            - grid-template-areas: "\"main\""
            - grid-template-columns: 1fr
          custom_fields:
            main:
              - width: 100%
              - min-width: 0
        custom_fields:
          main: |
            [[[
              const getState = entityId =>
                states[entityId]?.state;

              const getAttr = (entityId, attribute) =>
                states[entityId]?.attributes?.[attribute];

              const isBad = value =>
                value === undefined ||
                value === null ||
                [
                  'unknown',
                  'unavailable',
                  'none',
                  'null',
                  ''
                ].includes(
                  String(value).toLowerCase()
                );

              const isOn = entityId =>
                getState(entityId) === 'on';

              const formatNumber = (
                entityId,
                digits = 0,
                fallbackUnit = ''
              ) => {
                const stateObject =
                  states[entityId];

                if (
                  !stateObject ||
                  isBad(stateObject.state)
                ) {
                  return '—';
                }

                const value =
                  Number(stateObject.state);

                const unit =
                  stateObject.attributes
                    ?.unit_of_measurement ||
                  fallbackUnit;

                if (!Number.isFinite(value)) {
                  return stateObject.state;
                }

                return `${value.toFixed(digits)}${
                  unit ? ` ${unit}` : ''
                }`;
              };

              const battery = Number(
                getState(
                  'sensor.coaster_battery_level'
                )
              );

              const batterySafe =
                Number.isFinite(battery)
                  ? Math.max(
                      0,
                      Math.min(100, battery)
                    )
                  : 0;

              const range = formatNumber(
                'sensor.coaster_battery_range',
                0,
                'mi'
              );

              const chargeLimit = formatNumber(
                'number.coaster_charge_limit_2',
                0,
                '%'
              );

              const chargingState = String(
                getState(
                  'sensor.coaster_charging'
                ) || ''
              ).toLowerCase();

              const isCharging =
                ['charging', 'starting'].includes(chargingState) ||
                states['binary_sensor.coaster_charge_cable']?.state === 'on';

              const chargingComplete = [
                'complete',
                'completed',
                'charging complete'
              ].includes(chargingState);

              const plugged = isOn(
                'binary_sensor.coaster_charge_cable'
              );

              const scheduledCharging = isOn(
                'binary_sensor.coaster_scheduled_charging_pending'
              );

              const tripCharging = isOn(
                'binary_sensor.coaster_trip_charging'
              );

              const batteryHeater = isOn(
                'binary_sensor.coaster_battery_heater'
              );

              const vehicleImage =
                isCharging
                  ? '/local/Tesla/charging.png'
                  : '/local/Tesla/base.png';

              const userPresent = isOn(
                'binary_sensor.coaster_user_present_2'
              );

              const vehicleOnline = isOn(
                'binary_sensor.coaster_status'
              );

              const locked =
                getState(
                  'lock.coaster_lock'
                ) === 'locked';

              const sentry = isOn(
                'switch.coaster_sentry_mode_2'
              );

              const dashcam = isOn(
                'binary_sensor.coaster_dashcam'
              );

              const location = getState(
                'device_tracker.coaster_location'
              );

              let locationText =
                'Location unavailable';

              if (location === 'home') {
                locationText =
                  'Parked at home';
              } else if (
                location === 'not_home'
              ) {
                locationText =
                  'Away from home';
              } else if (!isBad(location)) {
                locationText =
                  String(location)
                    .replaceAll('_', ' ')
                    .replace(
                      /\b\w/g,
                      character =>
                        character.toUpperCase()
                    );
              }

              const speed = Number(
                getState(
                  'sensor.coaster_speed'
                )
              );

              const shiftState = String(
                getState(
                  'sensor.coaster_shift_state_2'
                ) || ''
              ).toUpperCase();

              const isDriving =
                (
                  Number.isFinite(speed) &&
                  speed > 0
                ) ||
                ['D', 'R', 'N'].includes(
                  shiftState
                );

              const hvacAction = String(
                getAttr(
                  'climate.coaster_climate',
                  'hvac_action'
                ) || ''
              ).toLowerCase();

              const climateRunning = [
                'heating',
                'cooling',
                'drying',
                'fan'
              ].includes(hvacAction);

              const cabinTemperature = Number(
                getAttr(
                  'climate.coaster_climate',
                  'current_temperature'
                )
              );

              const climateUnit =
                getAttr(
                  'climate.coaster_climate',
                  'temperature_unit'
                ) || '°';

              let statusTitle =
                'Parked';

              let statusSubtitle =
                vehicleOnline
                  ? 'Vehicle available'
                  : 'Vehicle asleep';

              let statusIcon =
                'mdi:car-electric';

              let accent =
                '#31dc69';

              if (isDriving) {
                statusTitle =
                  'Driving';

                statusSubtitle =
                  Number.isFinite(speed)
                    ? `${speed.toFixed(0)} mph`
                    : 'Vehicle in motion';

                statusIcon =
                  'mdi:car-speed-limiter';

                accent =
                  '#52a7ff';
              } else if (userPresent) {
                statusTitle =
                  'Vehicle occupied';

                statusSubtitle =
                  'Driver detected inside';

                statusIcon =
                  'mdi:account';

                accent =
                  '#52a7ff';
              } else if (isCharging) {
                statusTitle =
                  'Charging';

                statusSubtitle =
                  tripCharging
                    ? 'Trip charging active'
                    : 'Energy flowing into battery';

                statusIcon =
                  'mdi:battery-charging';

                accent =
                  '#31dc69';
              } else if (chargingComplete) {
                statusTitle =
                  'Charge complete';

                statusSubtitle =
                  'Battery reached target';

                statusIcon =
                  'mdi:battery-check';

                accent =
                  '#31dc69';
              } else if (
                scheduledCharging &&
                plugged
              ) {
                statusTitle =
                  'Charging scheduled';

                statusSubtitle =
                  'Waiting for scheduled start';

                statusIcon =
                  'mdi:calendar-clock';

                accent =
                  '#a979ff';
              } else if (climateRunning) {
                statusTitle =
                  hvacAction === 'heating'
                    ? 'Cabin heating'
                    : hvacAction === 'cooling'
                      ? 'Cabin cooling'
                      : 'Climate running';

                statusSubtitle =
                  'Cabin conditioning active';

                statusIcon =
                  hvacAction === 'heating'
                    ? 'mdi:heat-wave'
                    : 'mdi:snowflake';

                accent =
                  '#52a7ff';
              } else if (batteryHeater) {
                statusTitle =
                  'Battery heating';

                statusSubtitle =
                  'Battery conditioning active';

                statusIcon =
                  'mdi:heat-wave';

                accent =
                  '#ff9f43';
              }

              let batteryColor =
                '#31dc69';

              if (Number.isFinite(battery)) {
                if (battery <= 20) {
                  batteryColor =
                    '#ff5d5d';
                } else if (
                  battery <= 40
                ) {
                  batteryColor =
                    '#ff9f43';
                } else if (
                  battery <= 60
                ) {
                  batteryColor =
                    '#ffd84d';
                }
              }

              const chargeText =
                isCharging
                  ? 'Charging'
                  : chargingComplete
                    ? 'Charge complete'
                    : scheduledCharging &&
                        plugged
                      ? 'Scheduled'
                      : plugged
                        ? 'Connected'
                        : 'Not charging';

              const cabinText =
                Number.isFinite(
                  cabinTemperature
                )
                  ? `${cabinTemperature.toFixed(0)}${climateUnit}`
                  : '—';

              return `
                <div
                  class="hero-wrap ${
                    isCharging
                      ? 'is-charging'
                      : ''
                  }"
                  style="
                    --accent:${accent};
                    --battery-color:${batteryColor};
                  "
                >
                  <div class="hero-top">
                    <div>
                      <div class="vehicle-name">
                        Coaster
                      </div>

                      <div class="vehicle-meta">
                        <span class="status-dot"></span>

                        <span>
                          ${statusTitle}
                        </span>

                        <ha-icon
                          class="${
                            locked
                              ? ''
                              : 'unlocked-warning'
                          }"
                          icon="${
                            locked
                              ? 'mdi:lock'
                              : 'mdi:lock-open-alert'
                          }"
                        ></ha-icon>

                        <span class="${
                          locked
                            ? ''
                            : 'unlocked-warning'
                        }">
                          ${
                            locked
                              ? 'Locked'
                              : 'Unlocked'
                          }
                        </span>
                      </div>
                    </div>

                    <ha-icon
                      class="hero-menu"
                      icon="mdi:dots-horizontal"
                      style="cursor:pointer"
                      onclick="
                        event.preventDefault();
                        event.stopPropagation();
                        this.dispatchEvent(
                          new CustomEvent('hass-more-info', {
                            detail: {
                              entityId:
                                'binary_sensor.coaster_status'
                            },
                            bubbles: true,
                            composed: true
                          })
                        );
                      "
                    ></ha-icon>
                  </div>

                  <div class="hero-main">
                    <div class="battery-column">
                      <div class="battery-number">
                        ${
                          Number.isFinite(battery)
                            ? battery.toFixed(0)
                            : '—'
                        }

                        <small>%</small>

                        ${
                          isCharging
                            ? `
                              <ha-icon
                                icon="mdi:lightning-bolt"
                              ></ha-icon>
                            `
                            : ''
                        }
                      </div>

                      <div class="range-number">
                        ${range}
                      </div>

                      <div class="range-label">
                        Estimated range
                      </div>

                      <div class="hero-battery-track">
                        <div
                          class="hero-battery-fill ${
                            isCharging
                              ? 'charging'
                              : ''
                          }"
                          style="
                            width:${batterySafe}%;
                          "
                        >
                          ${
                            isCharging
                              ? `
                                <span
                                  class="charge-sweep"
                                ></span>
                              `
                              : ''
                          }
                        </div>
                      </div>

                      <div class="charge-limit">
                        Charge limit:
                        ${chargeLimit}
                      </div>

                      <div class="charge-state">
                        ${chargeText}
                      </div>
                    </div>

                    <div class="vehicle-image-wrap">
                      <img
                        src="${vehicleImage}"
                        class="vehicle-image ${
                          isCharging
                            ? 'charging-image'
                            : 'base-image'
                        }"
                      />
                    </div>
                  </div>

                  <div class="hero-feature-row">
                    <div
                      class="hero-feature sentry-toggle ${
                        sentry
                          ? 'feature-active'
                          : ''
                      }"
                      role="button"
                      tabindex="0"
                      title="${
                        sentry
                          ? 'Turn Sentry Mode off'
                          : 'Turn Sentry Mode on'
                      }"
                      onclick="
                        event.preventDefault();
                        event.stopPropagation();

                        const homeAssistant =
                          document.querySelector(
                            'home-assistant'
                          );

                        if (
                          homeAssistant &&
                          homeAssistant.hass
                        ) {
                          homeAssistant.hass.callService(
                            'switch',
                            'toggle',
                            {
                              entity_id:
                                'switch.coaster_sentry_mode_2'
                            }
                          );
                        }
                      "
                    >
                      <ha-icon
                        icon="${
                          sentry
                            ? 'phu:tesla-sentry'
                            : 'phu:tesla-sentry'
                        }"
                      ></ha-icon>

                      <div>
                        <span>
                          Sentry
                        </span>

                        <strong>
                          ${
                            sentry
                              ? 'On'
                              : 'Off'
                          }
                        </strong>
                      </div>

                      <i class="${
                        sentry
                          ? 'on'
                          : 'off'
                      }"></i>
                    </div>

                    <div class="feature-divider"></div>

                    <div
                      class="hero-feature"
                      role="button"
                      tabindex="0"
                      style="cursor:pointer"
                      title="Open dashcam details"
                      onclick="
                        event.preventDefault();
                        event.stopPropagation();
                        this.dispatchEvent(
                          new CustomEvent('hass-more-info', {
                            detail: {
                              entityId:
                                'binary_sensor.coaster_dashcam'
                            },
                            bubbles: true,
                            composed: true
                          })
                        );
                      "
                    >
                      <ha-icon
                        icon="${
                          dashcam
                            ? 'mdi:camera'
                            : 'mdi:camera-off'
                        }"
                      ></ha-icon>

                      <div>
                        <span>
                          Dashcam
                        </span>

                        <strong>
                          ${
                            dashcam
                              ? 'On'
                              : 'Off'
                          }
                        </strong>
                      </div>

                      <i class="${
                        dashcam
                          ? 'on'
                          : 'off'
                      }"></i>
                    </div>

                    <div class="feature-divider"></div>

                    <div
                      class="hero-feature"
                      role="button"
                      tabindex="0"
                      style="cursor:pointer"
                      title="Open climate"
                      onclick="
                        event.preventDefault();
                        event.stopPropagation();
                        this.dispatchEvent(
                          new CustomEvent('hass-more-info', {
                            detail: {
                              entityId:
                                'climate.coaster_climate'
                            },
                            bubbles: true,
                            composed: true
                          })
                        );
                      "
                    >
                      <ha-icon
                        icon="${
                          climateRunning
                            ? 'mdi:fan'
                            : 'mdi:fan-off'
                        }"
                      ></ha-icon>

                      <div>
                        <span>
                          Cabin
                        </span>

                        <strong>
                          ${cabinText}
                        </strong>
                      </div>

                      <i class="${
                        climateRunning
                          ? 'on'
                          : 'neutral'
                      }"></i>
                    </div>
                  </div>

                  <div class="hero-status-line">
                    <ha-icon
                      icon="${statusIcon}"
                    ></ha-icon>

                    <div>
                      <strong>
                        ${statusTitle}
                      </strong>

                      <span>
                        ${statusSubtitle}
                      </span>
                    </div>

                    <span class="hero-location">
                      ${locationText}
                    </span>
                  </div>
                </div>
              `;
            ]]]
        extra_styles: |
          @keyframes chargeSweep {
            0% {
              transform: translateX(-140%);
            }

            100% {
              transform: translateX(460%);
            }
          }

          @keyframes chargingGlow {
            0%,
            100% {
              opacity: 0.44;
              transform:
                translate(-50%, -50%)
                scale(0.95);
            }

            50% {
              opacity: 0.82;
              transform:
                translate(-50%, -50%)
                scale(1.05);
            }
          }

          @keyframes sentryTap {
            0% {
              transform: scale(1);
            }

            50% {
              transform: scale(0.95);
            }

            100% {
              transform: scale(1);
            }
          }

          .hero-wrap {
            width: 100%;
            min-width: 0;
            color: white;
            font-family:
              -apple-system,
              BlinkMacSystemFont,
              "Segoe UI",
              Roboto,
              sans-serif;
          }

          .hero-top {
            display: flex;
            align-items: flex-start;
            justify-content: space-between;
          }

          .vehicle-name {
            font-size: 27px;
            line-height: 1;
            font-weight: 800;
            letter-spacing: -0.8px;
          }

          .vehicle-meta {
            display: flex;
            align-items: center;
            gap: 7px;
            margin-top: 10px;
            color: rgba(255,255,255,0.64);
            font-size: 12px;
            font-weight: 650;
          }

          .vehicle-meta ha-icon {
            width: 13px;
            height: 13px;
            margin-left: 4px;
          }

          .unlocked-warning {
            color: #ff7068 !important;
          }

          .status-dot {
            width: 8px;
            height: 8px;
            flex: 0 0 8px;
            border-radius: 50%;
            background: var(--accent);
            box-shadow:
              0 0 11px
              color-mix(
                in srgb,
                var(--accent) 70%,
                transparent
              );
          }

          .hero-menu {
            width: 20px;
            height: 20px;
            color: rgba(255,255,255,0.76);
          }

          .hero-main {
            position: relative;
            display: grid;
            grid-template-columns:
              minmax(112px,0.84fr)
              minmax(0,1.16fr);
            min-height: 190px;
            margin-top: 12px;
            overflow: visible;
          }

          .battery-column {
            position: relative;
            z-index: 4;
            align-self: center;
            min-width: 0;
            text-align: left;
          }

          .battery-number {
            display: flex;
            align-items: center;
            color: white;
            font-size: 57px;
            line-height: 0.92;
            font-weight: 700;
            letter-spacing: -3px;
          }

          .battery-number small {
            margin-left: 4px;
            font-size: 23px;
            letter-spacing: -1px;
          }

          .battery-number ha-icon {
            width: 29px;
            height: 29px;
            margin-left: 7px;
            color: var(--battery-color);
            filter:
              drop-shadow(
                0 0 8px
                color-mix(
                  in srgb,
                  var(--battery-color) 60%,
                  transparent
                )
              );
          }

          .range-number {
            margin-top: 16px;
            font-size: 21px;
            line-height: 1;
            font-weight: 750;
          }

          .range-label {
            margin-top: 6px;
            color: rgba(255,255,255,0.49);
            font-size: 11px;
            font-weight: 600;
          }

          .hero-battery-track {
            position: relative;
            width: 115px;
            height: 8px;
            overflow: hidden;
            margin-top: 14px;
            border-radius: 999px;
            background: rgba(255,255,255,0.10);
          }

          .hero-battery-fill {
            position: relative;
            height: 100%;
            min-width: 3px;
            overflow: hidden;
            border-radius: inherit;
            background:
              linear-gradient(
                90deg,
                var(--battery-color),
                color-mix(
                  in srgb,
                  var(--battery-color) 76%,
                  white
                )
              );
            box-shadow:
              0 0 14px
              color-mix(
                in srgb,
                var(--battery-color) 48%,
                transparent
              );
            transition: width 0.6s ease;
          }

          .charge-sweep {
            position: absolute;
            inset: 0 auto 0 0;
            width: 38%;
            background:
              linear-gradient(
                90deg,
                transparent,
                rgba(255,255,255,0.78),
                transparent
              );
            animation:
              chargeSweep
              2s linear infinite;
          }

          .charge-limit,
          .charge-state {
            font-size: 11px;
            font-weight: 650;
          }

          .charge-limit {
            margin-top: 12px;
            color: rgba(255,255,255,0.65);
          }

          .charge-state {
            margin-top: 7px;
            color: rgba(255,255,255,0.86);
          }

          .vehicle-image-wrap {
            position: relative;
            display: flex;
            align-items: center;
            justify-content: flex-start;
            min-width: 0;
            overflow: visible;
            transform: translateX(-10px);
          }

          .vehicle-image-wrap::before {
            position: absolute;
            left: 34%;
            top: 50%;
            width: 165px;
            height: 122px;
            content: "";
            border-radius: 50%;
            background: rgba(39,235,99,0.12);
            filter: blur(33px);
            transform:
              translate(-50%, -50%);
            transition:
              background 0.4s ease,
              opacity 0.4s ease;
          }

          .is-charging
          .vehicle-image-wrap::before {
            left: 24%;
            background: rgba(49,220,105,0.21);
            animation:
              chargingGlow
              2.4s ease-in-out infinite;
          }

          .vehicle-image {
            position: relative;
            z-index: 2;
            display: block;
            width: 110%;
            max-width: 252px;
            height: auto;
            object-fit: contain;
            object-position: center;
            transform:
              translateX(-5px)
              translateY(5px);
            filter:
              drop-shadow(
                0 17px 15px
                rgba(0,0,0,0.58)
              );
            transition:
              opacity 0.35s ease,
              filter 0.35s ease,
              transform 0.35s ease;
          }

          .vehicle-image.charging-image {
            width: 110%;
            max-width: 252px;
          }

          .hero-feature-row {
            display: grid;
            grid-template-columns:
              1fr auto 1fr auto 1fr;
            align-items: stretch;
            min-height: 67px;
            padding: 0 9px;
            border-radius: 18px;
            background: rgba(255,255,255,0.035);
            border: 1px solid rgba(255,255,255,0.08);
          }

          .hero-feature {
            display: grid;
            grid-template-columns:
              25px minmax(0,1fr) 8px;
            gap: 7px;
            align-items: center;
            min-width: 0;
            padding: 8px 2px;
            text-align: left;
          }

          .sentry-toggle {
            cursor: pointer;
            border-radius: 13px;
            transition:
              background 0.2s ease,
              transform 0.12s ease;
            -webkit-tap-highlight-color: transparent;
          }

          .sentry-toggle:hover {
            background: rgba(255,255,255,0.045);
          }

          .sentry-toggle:active {
            animation:
              sentryTap
              0.2s ease;
            background: rgba(49,220,105,0.08);
          }

          .sentry-toggle.feature-active {
            background: rgba(49,220,105,0.035);
          }

          .hero-feature ha-icon {
            width: 21px;
            height: 21px;
            color: rgba(255,255,255,0.78);
          }

          .sentry-toggle.feature-active ha-icon {
            color: #31dc69;
            filter:
              drop-shadow(
                0 0 7px
                rgba(49,220,105,0.38)
              );
          }

          .hero-feature div {
            min-width: 0;
          }

          .hero-feature span,
          .hero-feature strong {
            display: block;
            overflow: hidden;
            text-overflow: ellipsis;
            white-space: nowrap;
          }

          .hero-feature span {
            color: rgba(255,255,255,0.48);
            font-size: 9px;
            font-weight: 650;
          }

          .hero-feature strong {
            margin-top: 4px;
            color: rgba(255,255,255,0.90);
            font-size: 11px;
            font-weight: 750;
          }

          .hero-feature i {
            width: 6px;
            height: 6px;
            border-radius: 50%;
          }

          .hero-feature i.on {
            background: #31dc69;
            box-shadow:
              0 0 8px #31dc69;
          }

          .hero-feature i.off {
            background: #ff5353;
            box-shadow:
              0 0 8px
              rgba(255,83,83,0.60);
          }

          .hero-feature i.neutral {
            background:
              rgba(255,255,255,0.28);
          }

          .feature-divider {
            align-self: center;
            width: 1px;
            height: 31px;
            margin: 0 8px;
            background:
              rgba(255,255,255,0.09);
          }

          .hero-status-line {
            display: grid;
            grid-template-columns:
              25px minmax(0,1fr) auto;
            gap: 9px;
            align-items: center;
            margin-top: 10px;
            padding: 8px 10px;
            border-radius: 15px;
            text-align: left;
            background:
              color-mix(
                in srgb,
                var(--accent) 7%,
                rgba(255,255,255,0.025)
              );
            border:
              1px solid
              color-mix(
                in srgb,
                var(--accent) 17%,
                rgba(255,255,255,0.06)
              );
          }

          .hero-status-line > ha-icon {
            width: 19px;
            height: 19px;
            color: var(--accent);
          }

          .hero-status-line div {
            min-width: 0;
          }

          .hero-status-line strong,
          .hero-status-line span {
            display: block;
            overflow: hidden;
            text-overflow: ellipsis;
            white-space: nowrap;
          }

          .hero-status-line strong {
            font-size: 11px;
            font-weight: 780;
          }

          .hero-status-line div span {
            margin-top: 3px;
            color: rgba(255,255,255,0.44);
            font-size: 9px;
          }

          .hero-location {
            max-width: 102px;
            color: rgba(255,255,255,0.56);
            font-size: 9px;
            font-weight: 650;
            text-align: right;
          }

          @media (max-width: 420px) {
            .hero-main {
              grid-template-columns:
                minmax(108px,0.88fr)
                minmax(0,1.12fr);
            }

            .vehicle-image-wrap {
              transform: translateX(-13px);
            }

            .vehicle-image,
            .vehicle-image.charging-image {
              width: 106%;
              max-width: 240px;
            }

            .vehicle-image-wrap::before {
              left: 30%;
              width: 154px;
              height: 114px;
            }

            .is-charging
            .vehicle-image-wrap::before {
              left: 19%;
            }
          }

          @media (max-width: 380px) {
            .hero-main {
              grid-template-columns:
                minmax(104px,0.92fr)
                minmax(0,1.08fr);
            }

            .vehicle-image-wrap {
              transform: translateX(-17px);
            }

            .vehicle-image,
            .vehicle-image.charging-image {
              width: 101%;
              max-width: 226px;
              transform:
                translateX(-3px)
                translateY(6px);
            }

            .vehicle-image-wrap::before {
              left: 26%;
              width: 143px;
              height: 106px;
            }

            .is-charging
            .vehicle-image-wrap::before {
              left: 15%;
            }

            .hero-feature-row {
              padding: 0 6px;
            }

            .feature-divider {
              margin: 0 4px;
            }

            .hero-feature {
              grid-template-columns:
                21px minmax(0,1fr) 6px;
              gap: 4px;
            }

            .hero-feature ha-icon {
              width: 18px;
              height: 18px;
            }
          }
      - type: custom:button-card
        show_name: false
        show_icon: false
        show_state: false
        triggers_update:
          - sensor.coaster_charger_power_3
          - sensor.coaster_charge_energy_added
          - sensor.coaster_charger_voltage
          - sensor.coaster_charger_current
          - sensor.coaster_charge_rate
          - sensor.coaster_charging
          - binary_sensor.coaster_charge_cable
          - binary_sensor.coaster_battery_heater
          - binary_sensor.coaster_trip_charging
        styles:
          card:
            - padding: 0
            - border-radius: 22px
            - overflow: hidden
            - background: |
                linear-gradient(
                  145deg,
                  rgba(14,25,37,0.97),
                  rgba(5,12,20,0.99)
                )
            - border: 1px solid rgba(255,255,255,0.09)
            - box-shadow: 0 12px 28px rgba(0,0,0,0.25)
          grid:
            - grid-template-areas: "\"main\""
            - grid-template-columns: 1fr
        custom_fields:
          main: |
            [[[
              const format = (
                entityId,
                digits = 0,
                fallbackUnit = ''
              ) => {
                const stateObject =
                  states[entityId];

                if (
                  !stateObject ||
                  [
                    'unknown',
                    'unavailable'
                  ].includes(
                    stateObject.state
                  )
                ) {
                  return '—';
                }

                let value =
                  Number(
                    stateObject.state
                  );

                let unit =
                  stateObject.attributes
                    ?.unit_of_measurement ||
                  fallbackUnit;

                if (
                  !Number.isFinite(value)
                ) {
                  return stateObject.state;
                }

                if (
                  entityId ===
                    'sensor.coaster_charger_power_3' &&
                  unit.toLowerCase() === 'w'
                ) {
                  value =
                    value / 1000;

                  unit =
                    'kW';
                }

                return `${value.toFixed(digits)}${
                  unit ? ` ${unit}` : ''
                }`;
              };

              const chargingState = String(
                states[
                  'sensor.coaster_charging'
                ]?.state || ''
              ).toLowerCase();

              const isCharging =
                ['charging', 'starting'].includes(chargingState) ||
                states['binary_sensor.coaster_charge_cable']?.state === 'on';

              const plugged =
                states[
                  'binary_sensor.coaster_charge_cable'
                ]?.state === 'on';

              const tripCharging =
                states[
                  'binary_sensor.coaster_trip_charging'
                ]?.state === 'on';

              const batteryHeater =
                states[
                  'binary_sensor.coaster_battery_heater'
                ]?.state === 'on';

              const power = format(
                'sensor.coaster_charger_power_3',
                1,
                'kW'
              );

              const energy = format(
                'sensor.coaster_charge_energy_added',
                1,
                'kWh'
              );

              const voltage = format(
                'sensor.coaster_charger_voltage',
                0,
                'V'
              );

              const current = format(
                'sensor.coaster_charger_current',
                0,
                'A'
              );

              const rate = format(
                'sensor.coaster_charge_rate',
                0,
                'mi/h'
              );

              let chargingLabel =
                'Not charging';

              if (isCharging) {
                chargingLabel =
                  tripCharging
                    ? 'Trip charging'
                    : 'Active charging';
              } else if (plugged) {
                chargingLabel =
                  'Cable connected';
              }

              return `
                <div class="stat-wrap">
                  <div class="stat-grid">
                    <div>
                      <span>
                        CHARGE POWER
                      </span>

                      <strong>
                        ${power}
                      </strong>
                    </div>

                    <div>
                      <span>
                        ADDED
                      </span>

                      <strong>
                        ${energy}
                      </strong>
                    </div>

                    <div>
                      <span>
                        VOLTAGE
                      </span>

                      <strong>
                        ${voltage}
                      </strong>
                    </div>

                    <div>
                      <span>
                        CURRENT
                      </span>

                      <strong>
                        ${current}
                      </strong>
                    </div>
                  </div>

                  <div class="charging-summary">
                    <div>
                      <ha-icon
                        icon="${
                          isCharging
                            ? 'mdi:battery-charging'
                            : plugged
                              ? 'mdi:power-plug'
                              : 'mdi:power-plug-off'
                        }"
                      ></ha-icon>

                      <span>
                        Charging
                      </span>
                    </div>

                    <strong>
                      ${chargingLabel}
                    </strong>
                  </div>

                  <div class="charging-summary">
                    <div>
                      <ha-icon
                        icon="mdi:speedometer"
                      ></ha-icon>

                      <span>
                        Charge rate
                      </span>
                    </div>

                    <strong>
                      ${rate}
                    </strong>
                  </div>

                  <div class="charging-summary">
                    <div>
                      <ha-icon
                        icon="mdi:heat-wave"
                      ></ha-icon>

                      <span>
                        Battery heater
                      </span>
                    </div>

                    <strong>
                      ${
                        batteryHeater
                          ? 'Running'
                          : 'Off'
                      }
                    </strong>
                  </div>
                </div>
              `;
            ]]]
        extra_styles: |
          .stat-wrap {
            color: white;
            font-family:
              -apple-system,
              BlinkMacSystemFont,
              "Segoe UI",
              Roboto,
              sans-serif;
          }

          .stat-grid {
            display: grid;
            grid-template-columns:
              repeat(
                4,
                minmax(0,1fr)
              );
            padding: 13px 8px;
          }

          .stat-grid > div {
            min-width: 0;
            padding: 0 6px;
            text-align: center;
            border-right:
              1px solid
              rgba(255,255,255,0.08);
          }

          .stat-grid > div:last-child {
            border-right: 0;
          }

          .stat-grid span,
          .stat-grid strong {
            display: block;
            overflow: hidden;
            text-overflow: ellipsis;
            white-space: nowrap;
          }

          .stat-grid span {
            color: rgba(255,255,255,0.43);
            font-size: 8px;
            font-weight: 750;
            letter-spacing: 0.5px;
          }

          .stat-grid strong {
            margin-top: 7px;
            color: rgba(255,255,255,0.92);
            font-size: 14px;
            font-weight: 800;
          }

          .charging-summary {
            display: flex;
            align-items: center;
            justify-content: space-between;
            min-height: 39px;
            padding: 0 14px;
            border-top:
              1px solid
              rgba(255,255,255,0.07);
          }

          .charging-summary div {
            display: flex;
            align-items: center;
            gap: 9px;
            min-width: 0;
          }

          .charging-summary ha-icon {
            width: 17px;
            height: 17px;
            color: rgba(255,255,255,0.57);
          }

          .charging-summary span {
            color: rgba(255,255,255,0.63);
            font-size: 11px;
            font-weight: 650;
          }

          .charging-summary strong {
            color: rgba(255,255,255,0.80);
            font-size: 11px;
            font-weight: 730;
          }
      - type: custom:button-card
        entity: climate.coaster_climate
        show_name: false
        show_icon: false
        show_state: false
        tap_action:
          action: more-info
        triggers_update:
          - climate.coaster_climate
          - binary_sensor.coaster_preconditioning_enabled
          - binary_sensor.coaster_cabin_overheat_protection
          - binary_sensor.coaster_cabin_overheat_protection_actively_cooling
          - binary_sensor.coaster_auto_seat_climate_left
          - binary_sensor.coaster_auto_seat_climate_right
          - binary_sensor.coaster_auto_steering_wheel_heater
        styles:
          card:
            - padding: 0
            - border-radius: 22px
            - background: |
                linear-gradient(
                  145deg,
                  rgba(14,25,37,0.97),
                  rgba(5,12,20,0.99)
                )
            - border: 1px solid rgba(255,255,255,0.09)
            - box-shadow: 0 12px 28px rgba(0,0,0,0.25)
          grid:
            - grid-template-areas: "\"main\""
            - grid-template-columns: 1fr
        custom_fields:
          main: |
            [[[
              const climate =
                states[
                  'climate.coaster_climate'
                ];

              const hvacAction = String(
                climate?.attributes
                  ?.hvac_action || ''
              ).toLowerCase();

              const climateRunning = [
                'heating',
                'cooling',
                'drying',
                'fan'
              ].includes(hvacAction);

              const currentTemperature =
                Number(
                  climate?.attributes
                    ?.current_temperature
                );

              const targetTemperature =
                Number(
                  climate?.attributes
                    ?.temperature
                );

              const unit =
                climate?.attributes
                  ?.temperature_unit ||
                '°';

              const preconditioningEnabled =
                states[
                  'binary_sensor.coaster_preconditioning_enabled'
                ]?.state === 'on';

              const overheatEnabled =
                states[
                  'binary_sensor.coaster_cabin_overheat_protection'
                ]?.state === 'on';

              const overheatCooling =
                states[
                  'binary_sensor.coaster_cabin_overheat_protection_actively_cooling'
                ]?.state === 'on';

              let title =
                'Climate';

              let subtitle =
                'Climate is off';

              let icon =
                'mdi:fan-off';

              if (overheatCooling) {
                title =
                  'Cabin cooling';

                subtitle =
                  'Overheat protection active';

                icon =
                  'mdi:snowflake';
              } else if (
                climateRunning
              ) {
                title =
                  hvacAction === 'heating'
                    ? 'Cabin heating'
                    : hvacAction === 'cooling'
                      ? 'Cabin cooling'
                      : 'Climate running';

                subtitle =
                  'Climate system active';

                icon =
                  hvacAction === 'heating'
                    ? 'mdi:heat-wave'
                    : 'mdi:fan';
              } else if (
                preconditioningEnabled
              ) {
                subtitle =
                  'Preconditioning enabled';
              }

              return `
                <div class="climate-wrap ${
                  climateRunning ||
                  overheatCooling
                    ? 'active'
                    : ''
                }">
                  <div class="climate-main">
                    <ha-icon
                      icon="${icon}"
                    ></ha-icon>

                    <div class="climate-current">
                      ${
                        Number.isFinite(
                          currentTemperature
                        )
                          ? `${currentTemperature.toFixed(0)}${unit}`
                          : '—'
                      }
                    </div>

                    <div class="climate-copy">
                      <strong>
                        ${title}
                      </strong>

                      <span>
                        ${subtitle}
                      </span>
                    </div>

                    <div class="climate-target">
                      ${
                        Number.isFinite(
                          targetTemperature
                        )
                          ? `${targetTemperature.toFixed(0)}${unit}`
                          : '—'
                      }
                    </div>
                  </div>

                  <div class="climate-pills">
                    <span class="${
                      preconditioningEnabled
                        ? 'enabled'
                        : ''
                    }">
                      Precondition ${
                        preconditioningEnabled
                          ? 'enabled'
                          : 'off'
                      }
                    </span>

                    <span class="${
                      overheatEnabled
                        ? 'enabled'
                        : ''
                    }">
                      Overheat ${
                        overheatEnabled
                          ? 'enabled'
                          : 'off'
                      }
                    </span>
                  </div>
                </div>
              `;
            ]]]
        extra_styles: |
          .climate-wrap {
            padding: 12px;
            color: white;
            font-family:
              -apple-system,
              BlinkMacSystemFont,
              "Segoe UI",
              Roboto,
              sans-serif;
          }

          .climate-main {
            display: grid;
            grid-template-columns:
              28px
              56px
              minmax(0,1fr)
              52px;
            gap: 8px;
            align-items: center;
          }

          .climate-main > ha-icon {
            width: 22px;
            height: 22px;
            color: rgba(255,255,255,0.72);
          }

          .climate-wrap.active
          .climate-main > ha-icon {
            color: #52a7ff;
            filter:
              drop-shadow(
                0 0 8px
                rgba(82,167,255,0.55)
              );
          }

          .climate-current,
          .climate-target {
            font-size: 20px;
            font-weight: 750;
            text-align: center;
          }

          .climate-copy {
            min-width: 0;
            text-align: center;
          }

          .climate-copy strong,
          .climate-copy span {
            display: block;
            overflow: hidden;
            text-overflow: ellipsis;
            white-space: nowrap;
          }

          .climate-copy strong {
            font-size: 12px;
            font-weight: 750;
          }

          .climate-copy span {
            margin-top: 5px;
            color: rgba(255,255,255,0.45);
            font-size: 9px;
            font-weight: 500;
          }

          .climate-pills {
            display: flex;
            justify-content: center;
            gap: 7px;
            margin-top: 10px;
          }

          .climate-pills span {
            padding: 5px 8px;
            border-radius: 999px;
            color: rgba(255,255,255,0.40);
            background: rgba(255,255,255,0.04);
            border: 1px solid rgba(255,255,255,0.06);
            font-size: 8px;
            font-weight: 700;
          }

          .climate-pills span.enabled {
            color: #52a7ff;
            background: rgba(82,167,255,0.08);
            border-color: rgba(82,167,255,0.14);
          }
      - type: grid
        columns: 3
        square: false
        cards:
          - type: custom:button-card
            entity: lock.coaster_lock
            name: Lock
            icon: mdi:lock
            show_state: false
            tap_action:
              action: toggle
            styles:
              card:
                - height: 88px
                - padding: 10px
                - border-radius: 20px
                - background: |
                    linear-gradient(
                      145deg,
                      rgba(18,29,41,0.97),
                      rgba(6,13,21,0.99)
                    )
                - border: 1px solid rgba(255,255,255,0.09)
                - box-shadow: 0 10px 24px rgba(0,0,0,0.22)
              grid:
                - grid-template-areas: "\"i\" \"n\""
                - grid-template-rows: 1fr auto
              icon:
                - width: 24px
                - color: rgba(255,255,255,0.82)
              name:
                - margin-top: 7px
                - font-size: 11px
                - font-weight: 700
                - color: rgba(255,255,255,0.88)
          - type: custom:button-card
            entity: button.coaster_flash_lights_2
            name: Flash
            icon: mdi:car-light-high
            show_state: false
            tap_action:
              action: call-service
              service: button.press
              target:
                entity_id: button.coaster_flash_lights_2
            styles:
              card:
                - height: 88px
                - padding: 10px
                - border-radius: 20px
                - background: |
                    linear-gradient(
                      145deg,
                      rgba(18,29,41,0.97),
                      rgba(6,13,21,0.99)
                    )
                - border: 1px solid rgba(255,255,255,0.09)
                - box-shadow: 0 10px 24px rgba(0,0,0,0.22)
              grid:
                - grid-template-areas: "\"i\" \"n\""
                - grid-template-rows: 1fr auto
              icon:
                - width: 24px
                - color: rgba(255,255,255,0.82)
              name:
                - margin-top: 7px
                - font-size: 11px
                - font-weight: 700
                - color: rgba(255,255,255,0.88)
          - type: custom:button-card
            entity: button.coaster_honk_horn
            name: Honk
            icon: mdi:bullhorn-outline
            show_state: false
            tap_action:
              action: call-service
              service: button.press
              target:
                entity_id: button.coaster_honk_horn
            styles:
              card:
                - height: 88px
                - padding: 10px
                - border-radius: 20px
                - background: |
                    linear-gradient(
                      145deg,
                      rgba(18,29,41,0.97),
                      rgba(6,13,21,0.99)
                    )
                - border: 1px solid rgba(255,255,255,0.09)
                - box-shadow: 0 10px 24px rgba(0,0,0,0.22)
              grid:
                - grid-template-areas: "\"i\" \"n\""
                - grid-template-rows: 1fr auto
              icon:
                - width: 24px
                - color: rgba(255,255,255,0.82)
              name:
                - margin-top: 7px
                - font-size: 11px
                - font-weight: 700
                - color: rgba(255,255,255,0.88)
          - type: custom:button-card
            entity: cover.coaster_frunk_2
            name: Frunk
            icon: phu:tesla-hood
            show_state: false
            tap_action:
              action: toggle
            confirmation:
              text: Open or close the frunk?
            styles:
              card:
                - height: 88px
                - padding: 10px
                - border-radius: 20px
                - background: |
                    linear-gradient(
                      145deg,
                      rgba(18,29,41,0.97),
                      rgba(6,13,21,0.99)
                    )
                - border: 1px solid rgba(255,255,255,0.09)
                - box-shadow: 0 10px 24px rgba(0,0,0,0.22)
              grid:
                - grid-template-areas: "\"i\" \"n\""
                - grid-template-rows: 1fr auto
              icon:
                - width: 24px
                - color: rgba(255,255,255,0.82)
              name:
                - margin-top: 7px
                - font-size: 11px
                - font-weight: 700
                - color: rgba(255,255,255,0.88)
          - type: custom:button-card
            entity: cover.coaster_trunk_2
            name: Trunk
            icon: phu:tesla-trunk
            show_state: false
            tap_action:
              action: toggle
            confirmation:
              text: Open or close the trunk?
            styles:
              card:
                - height: 88px
                - padding: 10px
                - border-radius: 20px
                - background: |
                    linear-gradient(
                      145deg,
                      rgba(18,29,41,0.97),
                      rgba(6,13,21,0.99)
                    )
                - border: 1px solid rgba(255,255,255,0.09)
                - box-shadow: 0 10px 24px rgba(0,0,0,0.22)
              grid:
                - grid-template-areas: "\"i\" \"n\""
                - grid-template-rows: 1fr auto
              icon:
                - width: 24px
                - color: rgba(255,255,255,0.82)
              name:
                - margin-top: 7px
                - font-size: 11px
                - font-weight: 700
                - color: rgba(255,255,255,0.88)
          - type: custom:button-card
            entity: cover.coaster_charge_port_door
            name: Charge Port
            icon: mdi:ev-plug-tesla
            show_state: false
            tap_action:
              action: toggle
            styles:
              card:
                - height: 88px
                - padding: 10px
                - border-radius: 20px
                - background: |
                    linear-gradient(
                      145deg,
                      rgba(18,29,41,0.97),
                      rgba(6,13,21,0.99)
                    )
                - border: 1px solid rgba(255,255,255,0.09)
                - box-shadow: 0 10px 24px rgba(0,0,0,0.22)
              grid:
                - grid-template-areas: "\"i\" \"n\""
                - grid-template-rows: 1fr auto
              icon:
                - width: 24px
                - color: rgba(255,255,255,0.82)
              name:
                - margin-top: 7px
                - font-size: 11px
                - font-weight: 700
                - color: rgba(255,255,255,0.88)
      - type: grid
        columns: 3
        square: false
        cards:
          - type: custom:button-card
            name: Doors
            icon: phu:tesla-door
            show_icon: true
            show_state: false
            show_label: true
            tap_action:
              action: more-info
              entity: binary_sensor.coaster_front_driver_door
            triggers_update:
              - binary_sensor.coaster_front_driver_door
              - binary_sensor.coaster_front_passenger_door
              - binary_sensor.coaster_rear_driver_door
              - binary_sensor.coaster_rear_passenger_door
            label: |
              [[[
                const entities = [
                  'binary_sensor.coaster_front_driver_door',
                  'binary_sensor.coaster_front_passenger_door',
                  'binary_sensor.coaster_rear_driver_door',
                  'binary_sensor.coaster_rear_passenger_door'
                ];

                const count =
                  entities.filter(
                    entityId =>
                      states[entityId]?.state ===
                      'on'
                  ).length;

                return count
                  ? `${count} open`
                  : 'All closed';
              ]]]
            styles:
              card:
                - min-height: 86px
                - padding: 10px
                - border-radius: 19px
                - background: |
                    linear-gradient(
                      145deg,
                      rgba(17,29,40,0.96),
                      rgba(7,14,22,0.99)
                    )
                - border: 1px solid rgba(255,255,255,0.08)
              grid:
                - grid-template-areas: "\"i n\" \"i l\""
                - grid-template-columns: 36px minmax(0,1fr)
                - grid-template-rows: auto auto
                - column-gap: 8px
              icon:
                - width: 25px
                - color: "#31dc69"
              name:
                - justify-self: start
                - font-size: 10px
                - font-weight: 750
                - color: rgba(255,255,255,0.88)
              label:
                - justify-self: start
                - margin-top: 4px
                - font-size: 8px
                - font-weight: 650
                - color: rgba(255,255,255,0.48)
          - type: custom:button-card
            name: Windows
            icon: mdi:car-door
            show_state: false
            show_label: true
            tap_action:
              action: more-info
              entity: binary_sensor.coaster_front_driver_window
            triggers_update:
              - binary_sensor.coaster_front_driver_window
              - binary_sensor.coaster_front_passenger_window
              - binary_sensor.coaster_rear_driver_window
              - binary_sensor.coaster_rear_passenger_window
            label: |
              [[[
                const entities = [
                  'binary_sensor.coaster_front_driver_window',
                  'binary_sensor.coaster_front_passenger_window',
                  'binary_sensor.coaster_rear_driver_window',
                  'binary_sensor.coaster_rear_passenger_window'
                ];

                const count =
                  entities.filter(
                    entityId =>
                      states[entityId]?.state ===
                      'on'
                  ).length;

                return count
                  ? `${count} open`
                  : 'All closed';
              ]]]
            styles:
              card:
                - min-height: 86px
                - padding: 10px
                - border-radius: 19px
                - background: |
                    linear-gradient(
                      145deg,
                      rgba(17,29,40,0.96),
                      rgba(7,14,22,0.99)
                    )
                - border: 1px solid rgba(255,255,255,0.08)
              grid:
                - grid-template-areas: "\"i n\" \"i l\""
                - grid-template-columns: 36px minmax(0,1fr)
                - grid-template-rows: auto auto
                - column-gap: 8px
              icon:
                - width: 25px
                - color: "#31dc69"
              name:
                - justify-self: start
                - font-size: 10px
                - font-weight: 750
                - color: rgba(255,255,255,0.88)
              label:
                - justify-self: start
                - margin-top: 4px
                - font-size: 8px
                - font-weight: 650
                - color: rgba(255,255,255,0.48)
          - type: custom:button-card
            name: Tire Pressure
            icon: mdi:car-tire-alert
            show_state: false
            show_label: true
            tap_action:
              action: more-info
              entity: binary_sensor.coaster_tire_pressure_warning_front_left
            triggers_update:
              - binary_sensor.coaster_tire_pressure_warning_front_left
              - binary_sensor.coaster_tire_pressure_warning_front_right
              - binary_sensor.coaster_tire_pressure_warning_rear_left
              - binary_sensor.coaster_tire_pressure_warning_rear_right
            label: |
              [[[
                const entities = [
                  'binary_sensor.coaster_tire_pressure_warning_front_left',
                  'binary_sensor.coaster_tire_pressure_warning_front_right',
                  'binary_sensor.coaster_tire_pressure_warning_rear_left',
                  'binary_sensor.coaster_tire_pressure_warning_rear_right'
                ];

                const count =
                  entities.filter(
                    entityId =>
                      states[entityId]?.state ===
                      'on'
                  ).length;

                return count
                  ? `${count} warning`
                  : 'Good';
              ]]]
            styles:
              card:
                - min-height: 86px
                - padding: 10px
                - border-radius: 19px
                - background: |
                    linear-gradient(
                      145deg,
                      rgba(17,29,40,0.96),
                      rgba(7,14,22,0.99)
                    )
                - border: 1px solid rgba(255,255,255,0.08)
              grid:
                - grid-template-areas: "\"i n\" \"i l\""
                - grid-template-columns: 36px minmax(0,1fr)
                - grid-template-rows: auto auto
                - column-gap: 8px
              icon:
                - width: 25px
                - color: "#31dc69"
              name:
                - justify-self: start
                - font-size: 10px
                - font-weight: 750
                - color: rgba(255,255,255,0.88)
              label:
                - justify-self: start
                - margin-top: 4px
                - font-size: 8px
                - font-weight: 650
                - color: rgba(255,255,255,0.48)
          - type: custom:button-card
            entity: binary_sensor.coaster_charge_cable
            name: Charge Cable
            icon: mdi:ev-plug-tesla
            show_state: false
            show_label: true
            label: |
              [[[
                return entity.state === 'on'
                  ? 'Connected'
                  : 'Disconnected';
              ]]]
            styles:
              card:
                - min-height: 86px
                - padding: 10px
                - border-radius: 19px
                - background: |
                    linear-gradient(
                      145deg,
                      rgba(17,29,40,0.96),
                      rgba(7,14,22,0.99)
                    )
                - border: 1px solid rgba(255,255,255,0.08)
              grid:
                - grid-template-areas: "\"i n\" \"i l\""
                - grid-template-columns: 36px minmax(0,1fr)
                - grid-template-rows: auto auto
                - column-gap: 8px
              icon:
                - width: 25px
                - color: "#31dc69"
              name:
                - justify-self: start
                - font-size: 10px
                - font-weight: 750
                - color: rgba(255,255,255,0.88)
              label:
                - justify-self: start
                - margin-top: 4px
                - font-size: 8px
                - font-weight: 650
                - color: rgba(255,255,255,0.48)
          - type: custom:button-card
            entity: binary_sensor.coaster_user_present_2
            name: User Present
            icon: mdi:account-outline
            show_state: false
            show_label: true
            label: |
              [[[
                return entity.state === 'on'
                  ? 'Yes'
                  : 'No';
              ]]]
            styles:
              card:
                - min-height: 86px
                - padding: 10px
                - border-radius: 19px
                - background: |
                    linear-gradient(
                      145deg,
                      rgba(17,29,40,0.96),
                      rgba(7,14,22,0.99)
                    )
                - border: 1px solid rgba(255,255,255,0.08)
              grid:
                - grid-template-areas: "\"i n\" \"i l\""
                - grid-template-columns: 36px minmax(0,1fr)
                - grid-template-rows: auto auto
                - column-gap: 8px
              icon:
                - width: 25px
                - color: "#31dc69"
              name:
                - justify-self: start
                - font-size: 10px
                - font-weight: 750
                - color: rgba(255,255,255,0.88)
              label:
                - justify-self: start
                - margin-top: 4px
                - font-size: 8px
                - font-weight: 650
                - color: rgba(255,255,255,0.48)
          - type: custom:button-card
            entity: binary_sensor.coaster_trip_charging
            name: Trip Charging
            icon: mdi:road-variant
            show_state: false
            show_label: true
            label: |
              [[[
                return entity.state === 'on'
                  ? 'Active'
                  : 'Inactive';
              ]]]
            styles:
              card:
                - min-height: 86px
                - padding: 10px
                - border-radius: 19px
                - background: |
                    linear-gradient(
                      145deg,
                      rgba(17,29,40,0.96),
                      rgba(7,14,22,0.99)
                    )
                - border: 1px solid rgba(255,255,255,0.08)
              grid:
                - grid-template-areas: "\"i n\" \"i l\""
                - grid-template-columns: 36px minmax(0,1fr)
                - grid-template-rows: auto auto
                - column-gap: 8px
              icon:
                - width: 25px
                - color: "#31dc69"
              name:
                - justify-self: start
                - font-size: 10px
                - font-weight: 750
                - color: rgba(255,255,255,0.88)
              label:
                - justify-self: start
                - margin-top: 4px
                - font-size: 8px
                - font-weight: 650
                - color: rgba(255,255,255,0.48)
