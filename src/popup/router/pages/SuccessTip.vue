<template>
  <div class="popup" data-cy="success-tip">
    <h3 class="heading-1 mb-25 mt-15 center">
      <div class="flex flex-align-center flex-justify-content-center">
        <Heart />
        <span class="ml-5">{{ $t('pages.successTip.completedHeading') }}</span>
      </div>
    </h3>
    <p class="primary-title text-left mb-8 f-16">
      {{ $t('pages.successTip.successfullySent') }}
      <span class="secondary-text" data-cy="tip-amount"
        >{{ amountTip }} {{ $t('pages.appVUE.aeid') }}
      </span>
      <!--eslint-disable-line vue-i18n/no-raw-text-->
      ({{ (amountTip * current.currencyRate).toFixed(3) }}
      {{ currentCurrency }})<!--eslint-disable-line vue-i18n/no-raw-text-->
      {{ $t('pages.successTip.to') }}
    </p>
    <a class="link-sm text-left block" data-cy="tip-url">{{ tipUrl }}</a>
    <br />
    <div>
      <span style="word-break: break-word; font-size: 14px; float: left;">{{
        $t('pages.successTip.notify')
      }}</span>
      <Textarea v-model="note" :value="note" size="h-50" />
    </div>
    <p class="f-14 sub-heading text-left" v-if="!(tipUrl && verifiedUrls.includes(tipUrl))">
      {{ $t('pages.successTip.note') }}
    </p>
    <p class="f-18 my-35">{{ $t('pages.successTip.letThemKnow') }}</p>
    <div>
      <div class="flex flex-align-center flex-justify-between">
        <Button half @click="$router.push('/tip')" data-cy="to-tips">
          {{ $t('pages.successTip.sendMore') }}
        </Button>
        <Button half @click="$router.push('/account')" data-cy="to-dashboard">
          {{ $t('pages.successTip.home') }}
        </Button>
      </div>
      <br />
      <Button @click="redirectOnFeed" extend>
        {{ $t('pages.successTip.feed') }}
      </Button>
    </div>
  </div>
</template>

<script>
import { mapGetters, mapState } from 'vuex';
import axios from 'axios';
import Heart from '../../../icons/heart.svg?vue-component';
import Textarea from '../components/Textarea';
import openUrl from '../../utils/openUrl';
import { TIP_SERVICE, BACKEND_URL, UI_URL } from '../../utils/constants';
import { aettosToAe } from '../../utils/helper';
import Logger from '../../../lib/logger';

export default {
  components: {
    Heart,
    Textarea,
  },
  props: ['amount', 'tipUrl'],
  data() {
    return {
      note: this.$t('pages.successTip.notifyMessage'),
      verifiedUrls: [],
    };
  },
  computed: {
    ...mapState(['current']),
    ...mapGetters(['currentCurrency']),
    amountTip() {
      return (+aettosToAe(this.amount)).toFixed(2);
    },
  },
  async created() {
    this.verifiedUrls = (
      await axios.get(`${BACKEND_URL}/verified`).catch(error => {
        Logger.write({ ...error, modal: false });
        return { data: [] };
      })
    ).data;
    if (process.env.IS_EXTENSION) {
      const { addresses, tab } = await this.$store.dispatch('getWebPageAddresses');
      if (addresses.length) {
        await axios
          .post(TIP_SERVICE, { url: tab.url, address: addresses[0] })
          .catch(error => Logger.write({ ...error, modal: false }));
      }
    }
  },
  methods: {
    redirectOnFeed() {
      openUrl(UI_URL);
    },
  },
};
</script>
