<i18n lang="yaml">
en:
  title: Reservation
  ggd_consent: I consent to share my information with the municipal health service (GGD) to support contact tracing if
    requested.
  seats:
    one_person_seat: Individual seat
    two_person_seat: Two-person seat
    within_distance: (within 1.5 metre)
    unavailable: No longer available
  event: Event
  no_events: There are no events scheduled at the moment. Please come back later.
  seat_type: Type of Seat
  thank_you:
    Thank you for making a reservation! You will receive an e-mail with your proof of reservation. If you need to
    cancel, you can do so via the e-mail you will receive.
  another_booking: Make another booking
  disclaimer:
    By submitting this form you allow DWH to save your information and to process it. Your information will only be
    used to check reservations and for the possibility to get in touch with you. If you have more questions, please
    contact us via the information below.
  oops: Oops.
  validation:
    A_guest_can_only_have_one_open_booking: You already have an active reservation for the coming days.
      You can only have one open booking at a time.
nl:
  title: Reserveren
  ggd_consent:
    Ik geef toestemming om mijn gegevens te delen met de GGD wanneer deze worden opgevraagd ten behoeve van een
    bron- en contactonderzoek.
  seats:
    one_person_seat: Eenpersoonszitplaats
    two_person_seat: Tweepersoonszitplaats
    within_distance: (binnen 1,5 meter)
    unavailable: Niet meer beschikbaar
  event: Evenement
  no_events: Er zijn op dit moment geen evenementen gepland. Kom later terug.
  seat_type: Type Zitplek
  thank_you:
    Bedankt voor het reserveren! Je ontvangt een e-mail met je reserveringsbewijs. Lukt het je uiteindelijk toch
    niet om aanwezig te zijn? Via de mail die je ontvangt kun je je reservering annuleren.
  another_booking: Maak nog een reservering
  disclaimer: Bij het verzenden van dit formulier geef je DWH toestemming om je gegevens op te slaan en te verwerken. Je
    gegevens worden alleen gebruikt voor het controleren van reserveringen en voor de mogelijkheid om contact
    met je op te nemen. Neem voor vragen contact op via onderstaande contactgegevens.
  oops: Oeps.
  validation:
    A_guest_can_only_have_one_open_booking:
      Je hebt al een actieve reservering voor een event de komende tijd. Je kunt maximaal 1 openstaande
      reservering hebben.
</i18n>

<template>
  <div class="bg-white p-8 rounded-lg shadow-xl">
    <div class="rounded-full w-16 h-16 p-5 bg-brand-450 mb-8 text-white">
      <Zondicon icon="time" class="fill-current" />
    </div>

    <h2 class="text-xl font-bold mb-4 text-brand-450 uppercase tracking-wider" v-text="$t('title')" />

    <div v-show="state === 'finished'">
      <FormCompleted class="bg-brand-100 mb-6" :title="$t('forms.success.heading')" :subtitle="$t('thank_you')" />
      <button
        class="border border-brand-500 hover:bg-brand-500 text-brand-500 hover:text-white rounded uppercase text-sm tracking-wide py-2 px-3 shadow"
        @click="reset"
        v-text="$t('another_booking')"
      />
    </div>

    <form v-show="state == 'start' || state == 'loading'" @submit="submit">
      <div v-for="error in errors" :key="error" class="bg-red-200 border-1 border-red-400 rounded p-2 mb-4">
        <strong>{{ $t('oops') }}</strong>
        {{ error }}
      </div>

      <FormElement :label="$t('forms.label.name')" class="form-element-gray" required="true">
        <FormInput v-model="form.name" :placeholder="$t('forms.placeholder.name')" />
      </FormElement>

      <FormElement :label="$t('forms.label.email')" class="form-element-gray" required="true">
        <FormInput v-model="form.email" :placeholder="$t('forms.placeholder.email')" type="email" />
      </FormElement>

      <FormCheckbox v-model="form.ggd_consent" :label="$t('ggd_consent')" />

      <FormElement v-show="form.ggd_consent" :label="$t('forms.label.phone_number')" class="form-element-gray">
        <FormInput v-model="form.phone_number" :placeholder="$t('forms.placeholder.phone_number')" />
      </FormElement>

      <div class="form-element-gray">
        <label class="required" v-text="$t('event')" />
        <div class="flex flex-wrap">
          <div v-if="events.length === 0" class="bg-brand-100 rounded-lg w-full p-4">
            {{ $t('no_events') }}
          </div>
          <div v-for="event in events" :key="event.id" class="w-full md:w-1/3">
            <div
              :class="[
                form.event_id == event.id ? 'border-2 border-brand-450' : '',
                event.available_seats == 0 ? 'text-gray-400' : 'cursor-pointer',
              ]"
              class="bg-brand-100 rounded-md mt-2 mr-2 shadow overflow-hidden"
              @click="selectEvent(event)"
            >
              <div
                :class="event.available_seats <= 0 ? 'bg-brand-200 text-brand-300' : 'text-white bg-brand-450'"
                class="px-3 py-1 font-bold"
                v-text="event.name"
              />
              <div class="px-3 py-2">
                <div class="capitalize mb-1 font-sembold" v-text="formatDate(event.start)" />
                <div class="bg-white px-2 py-1 rounded inline-flex items-center">
                  <Zondicon icon="time" class="fill-current w-4 h-4 mr-2" />
                  {{ formatTime(event.start) }} - {{ formatTime(event.end) }}
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>

      <div v-show="selectedEvent.id" class="form-element-gray">
        <label class="required" v-text="$t('seat_type')" />
        <div class="md:flex text-center">
          <div
            :class="form.twoseat ? '' : 'border-2 border-brand-450'"
            class="flex-1 md:mr-1 mb-2 md:mb-0 h-16 shadow bg-brand-100 rounded overflow-hidden cursor-pointer"
            @click="selectTwoseat(false)"
          >
            <div class="flex h-full">
              <div class="px-4 bg-brand-450 flex flex-col justify-center">
                <div class="rounded-full w-6 h-6 bg-white text-brand-450 p-1">
                  <Zondicon v-show="!form.twoseat" icon="checkmark" class="fill-current" />
                </div>
              </div>
              <div class="py-2 flex-1 flex flex-col justify-center" v-text="$t('seats.one_person_seat')" />
            </div>
          </div>
          <div
            :class="[
              form.twoseat ? 'border-2 border-brand-450' : '',
              selectedEvent.available_twoseats > 0 ? 'cursor-pointer' : '',
            ]"
            class="flex-1 md:ml-1 h-16 shadow bg-brand-100 rounded overflow-hidden"
            @click="selectTwoseat(true)"
          >
            <div class="flex h-full">
              <div
                :class="selectedEvent.available_twoseats > 0 ? 'bg-brand-450' : 'bg-brand-200'"
                class="px-4 flex flex-col justify-center"
              >
                <div class="rounded-full w-6 h-6 bg-white text-brand-450 p-1">
                  <Zondicon v-show="form.twoseat" icon="checkmark" class="fill-current" />
                </div>
              </div>
              <div class="py-2 flex-1 flex flex-col justify-center">
                {{ $t('seats.two_person_seat') }}
                <div
                  v-show="selectedEvent.available_twoseats > 0"
                  class="text-sm"
                  v-text="$t('seats.within_distance')"
                />
                <div v-show="selectedEvent.available_twoseats <= 0" class="text-sm" v-text="$t('seats.unavailable')" />
              </div>
            </div>
          </div>
        </div>
      </div>

      <div v-show="selectedEvent.event_type_id === 'dinner'" class="form-element-gray">
        <label class="required">Team</label>
        <div class="md:flex flex-wrap -mx-1">
          <div v-for="team in teams" :key="team.name" class="md:w-1/2 p-1">
            <div
              :class="[
                team === dinnerForm.team ? 'bg-brand-400 text-white' : 'bg-brand-100 hover:bg-brand-200',
                'rounded px-3 py-2 tracking-wider flex items-center cursor-pointer shadow',
              ]"
              @click="dinnerForm.team = team"
            >
              <span class="mr-3">{{ team.emoji }}</span>
              {{ team.name }}
            </div>
          </div>
        </div>
      </div>

      <div v-show="selectedEvent.event_type_id === 'dinner'" class="form-element">
        <label class="form-element-label">I don't eat...</label>
        <div class="md:flex">
          <div class="md:w-1/2 flex flex-wrap -m-1">
            <div
              v-for="restriction in restrictions"
              :key="restriction"
              :class="[
                dinnerForm.diet.includes(restriction) ? 'bg-brand-400 text-white' : 'bg-brand-100 hover:bg-brand-200',
                'rounded px-3 py-1 tracking-wider flex-1 text-center m-1 cursor-pointer shadow',
              ]"
              @click="toggleRestriction(restriction)"
              v-text="restriction"
            />
          </div>
          <div class="flex-1 md:w-1/2 mt-2 md:mt-0 md:ml-2">
            <div
              :class="[
                dinnerForm.otherDiet.length > 0 ? 'bg-brand-400 text-white' : 'bg-brand-100 hover:bg-brand-200',
                'rounded px-1 py-2 tracking-wider flex-1 flex items-center cursor-pointer h-full shadow',
              ]"
            >
              <span class="px-2">Other:</span>
              <input
                v-model="dinnerForm.otherDiet"
                type="text"
                class="mr-2 inline text-black"
                placeholder="Other Restrictions"
              />
            </div>
          </div>
        </div>
      </div>

      <p class="text-sm mb-4" v-text="$t('disclaimer')" />

      <PrimaryButton :disabled="state === 'loading'" type="submit">
        {{ state === 'loading' ? $t('forms.buttons.loading') : $t('forms.buttons.submit') }}
      </PrimaryButton>
    </form>
  </div>
</template>

<script>
import Zondicon from 'vue-zondicons'
import axios from 'axios'
import dayjs from 'dayjs'
import 'dayjs/locale/nl'

const apiUrl = 'https://reserveer.dwhdelft.nl'

const DINNER_TEAMS = [
  { emoji: '🍝', name: 'Kitchen Helpers' },
  { emoji: '⏰', name: 'In a hurry' },
  { emoji: '🧽 ', name: 'Washing up' },
  { emoji: '👩‍🍳', name: 'Cookies' },
  { emoji: '🍻', name: 'Bartender' },
]
const DINNER_RESTRICTIONS = ['Meat', 'Fish', 'Seafood', 'Cheese', 'Nuts', 'Dairy']

export default {
  components: { Zondicon },
  data() {
    return {
      errors: [],
      form: {
        name: '',
        email: '',
        ggd_consent: false,
        phone_number: '',
        event_id: undefined,
        twoseat: false,
        custom_fields: {},
      },
      events: [],
      selectedEvent: { available_twoseats: -1 },
      state: 'start',
      teams: DINNER_TEAMS,
      restrictions: DINNER_RESTRICTIONS,
      dinnerForm: {
        team: DINNER_TEAMS[0],
        diet: [],
        otherDiet: '',
      },
    }
  },
  mounted() {
    this.initialize()
  },
  methods: {
    initialize() {
      this.restorePersonalDetails()

      axios
        .get(apiUrl + '/api/events')
        .then((response) => {
          this.events = response.data.data
        })
        .catch(() => {
          alert('An error occurred. If this keeps happening, please send us an email.')
        })
    },
    selectEvent(event) {
      if (event.available_seats > 0) {
        this.form.event_id = event.id
        this.selectedEvent = event
      }
    },
    selectTwoseat(twoseat) {
      if (twoseat && this.selectedEvent.available_twoseats > 0) {
        this.form.twoseat = true
      } else {
        this.form.twoseat = false
      }
    },
    submit(event) {
      event.preventDefault()

      if (this.form.event_id === undefined) {
        this.errors.push('You have to select one of the timeslots.')
        return
      }

      if (this.selectedEvent.event_type_id === 'dinner') {
        this.form.custom_fields = {
          team: this.dinnerForm.team.name,
          diet: [...this.dinnerForm.diet, ...(this.dinnerForm.otherDiet.length > 0 ? [this.dinnerForm.otherDiet] : [])],
        }
      }

      this.state = 'loading'

      axios
        .post(apiUrl + '/api/bookings', this.form)
        .then(() => {
          this.state = 'finished'
          this.$el.scrollIntoView({ behavior: 'smooth' })
          this.cachePersonalDetails()
        })
        .catch((error) => {
          this.state = 'start'

          if (error.response.data.errors) {
            this.errors = Object.values(error.response.data.errors).flat().map(this.translateError)
          } else {
            alert('An error occurred. If this keeps happening, please send us an email.')
          }
        })
    },
    formatDate(date) {
      if (this.$i18n.locale === 'nl') {
        dayjs.locale('nl')
      }

      return dayjs(date).format('dddd DD-MM')
    },
    formatTime(date) {
      if (this.$i18n.locale === 'nl') {
        dayjs.locale('nl')
        return dayjs(date).format('H:mm')
      }

      return dayjs(date).format('h:mm A')
    },
    translateError(error) {
      const key = error.replace(/ /g, '_').replace(/\./g, '')

      if (this.$te('validation.' + key)) {
        return this.$t('validation.' + key)
      }

      return error
    },
    toggleRestriction(restriction) {
      if (this.dinnerForm.diet.includes(restriction)) {
        this.dinnerForm.diet.splice(this.dinnerForm.diet.indexOf(restriction), 1)
      } else {
        this.dinnerForm.diet.push(restriction)
      }
    },
    cachePersonalDetails() {
      window.localStorage.setItem(
        'booking_details',
        JSON.stringify({
          name: this.form.name,
          email: this.form.email,
          ggd_consent: this.form.ggd_consent,
          phone_number: this.form.phone_number,
        })
      )
    },
    restorePersonalDetails() {
      const cachedDetails = JSON.parse(window.localStorage.getItem('booking_details'))
      if (cachedDetails) {
        this.form.name = cachedDetails.name
        this.form.email = cachedDetails.email
        this.form.ggd_consent = cachedDetails.ggd_consent
        this.form.phone_number = cachedDetails.phone_number
      }
    },
    reset() {
      Object.assign(this.$data, this.$options.data.apply(this))
      this.initialize()
    },
  },
}
</script>
