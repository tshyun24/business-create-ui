<template>
  <div id="amalgamation-regular-information">
    <!-- Amalgamating Businesses -->
    <section class="mt-10">
      <header id="name-header">
        <h2>Amalgamating Businesses</h2>
        <p class="mt-4">
          Add each amalgamating business below.
        </p>
        <p class="mt-4">
          Your BC Registries account must have access to each business involved in this amalgamation
          before the amalgamation application can be filed.
        </p>
      </header>

      [*** TODO: Amalgamating Businesses component ***]
    </section>

    <!-- Resulting Business Name and Type -->
    <section class="mt-10">
      <header id="name-header">
        <h2>Resulting Business Name and Type</h2>
      </header>

      <ExpandableHelp
        class="mt-4"
        helpLabel="Help with Business Type"
      >
        <template #content>
          [*** TODO: help text component ***]
        </template>
      </ExpandableHelp>

      <v-card
        flat
        class="mt-5"
      >
        <NameRequestInfo />
        <NameTranslations class="mt-n8" />
      </v-card>
    </section>
  </div>
</template>

<script lang="ts">
import { Component, Mixins, Watch } from 'vue-property-decorator'
import { Getter, Action } from 'pinia-class'
import { useStore } from '@/store/store'
import { DefineCompanyIF } from '@/interfaces'
import { CommonMixin } from '@/mixins'
import { RouteNames } from '@/enums'
import { ExpandableHelp } from '@bcrs-shared-components/expandable-help'
import NameRequestInfo from '@/components/common/NameRequestInfo.vue'
import NameTranslations from '@/components/common/NameTranslations.vue'

@Component({
  components: {
    ExpandableHelp,
    NameRequestInfo,
    NameTranslations
  }
})
export default class AmalgamationRegularInformation extends Mixins(CommonMixin) {
  @Getter(useStore) getDefineCompanyStep!: DefineCompanyIF
  @Getter(useStore) getNameTranslationsValid!: boolean
  @Getter(useStore) getShowErrors!: boolean
  @Getter(useStore) isBaseCompany!: boolean

  @Action(useStore) setDefineCompanyStepValidity!: (x: boolean) => void
  @Action(useStore) setIgnoreChanges!: (x: boolean) => void

  // Local properties
  amalgamatingBusinessesValid = false

  /** Array of valid components. Must match validFlags below. */
  readonly validComponents = [
    'amalgamating-businesses',
    'name-header'
  ]

  /** Object of valid flags. Must match validComponents above. */
  get validFlags (): object {
    return {
      validAmalgamatingBusinesses: this.amalgamatingBusinessesValid,
      validNameTranslation: this.getNameTranslationsValid
    }
  }

  /** Called when component is created. */
  created (): void {
    // ignore data changes while page loads
    this.setIgnoreChanges(true)

    // watch data changes once page has loaded (in next tick)
    this.$nextTick(() => {
      this.setIgnoreChanges(false)
    })
  }

  onAmalgamatingBusinessesValid (valid: boolean): void {
    this.amalgamatingBusinessesValid = valid
    this.setDefineCompanyStepValidity(
      this.amalgamatingBusinessesValid &&
      this.getNameTranslationsValid
    )
  }

  @Watch('getNameTranslationsValid', { deep: true })
  private onNameTranslationsValid (): void {
    this.setDefineCompanyStepValidity(
      this.amalgamatingBusinessesValid &&
      this.getNameTranslationsValid
    )
  }

  @Watch('$route')
  private async scrollToInvalidComponent (): Promise<void> {
    if (this.getShowErrors && this.$route.name === RouteNames.AMALG_REG_INFORMATION) {
      // scroll to invalid components
      await this.$nextTick()
      await this.validateAndScroll(this.validFlags, this.validComponents)
    }
  }
}
</script>

<style lang="scss" scoped>
@import '@/assets/styles/theme.scss';

#amalgamation-regular-information {
  /* Set "header-counter" to 0 */
  counter-reset: header-counter;
}

h2::before {
  /* Increment "header-counter" by 1 */
  counter-increment: header-counter;
  content: counter(header-counter) '. ';
}

.value.name-request {
  width: 100%;
  min-width: 24rem;
}

.meta-container {
  display: flex;
  flex-flow: column nowrap;
  position: relative;

  > label:first-child {
    font-weight: bold;
  }
}

@media (min-width: 768px) {
  .meta-container {
    flex-flow: row nowrap;

    > label:first-child {
      flex: 0 0 auto;
      padding-right: 2rem;
      width: 12rem;
    }
  }
}

// #business-buttons-container {
//   .v-btn + .v-btn {
//     margin-left: 0.5rem;
//   }
// }

header p {
  padding-top: 0.5rem;
}

// Vuetify Overrides
:deep(.v-select__selection--comma) {
  overflow: inherit;
}
</style>
