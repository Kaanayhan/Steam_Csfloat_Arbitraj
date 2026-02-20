<template>
  <div class="wrap">
    <div class="top">
      <div>
        <h1>{{ t("title") }}</h1>
        <p class="sub">{{ t("subtitle") }}</p>
      </div>

      <div class="actions">
        <button @click="toggleLang">{{ currentLang }}</button>
        <button class="primary" @click="resetAll">{{ t("reset") }}</button>
      </div>
    </div>

    <div class="card cardRow">
      <h2>{{ t("startingBalances") }}</h2>
      <div class="form">
        <label>
          {{ t("initialSteam") }}
          <input type="number" step="0.01" v-model.number="s.initialSteam" />
        </label>

        <label>
          {{ t("initialCsfloat") }}
          <input type="number" step="0.01" v-model.number="s.initialCsfloat" />
        </label>
      </div>
    </div>

    <div class="grid">
      <div class="card">
        <h2>{{ t("leg1Title") }}</h2>

        <div class="form">
          <label>
            {{ t("quantity") }}
            <input type="number" step="1" v-model.number="s.leg1.qty" />
          </label>

          <label>
            {{ t("steamBuy") }}
            <input type="number" step="0.01" v-model.number="s.leg1.steamBuyPerItem" />
          </label>

          <label>
            {{ t("csfloatNet") }}
            <input type="number" step="0.01" v-model.number="s.leg1.csfloatNetPerItem" />
          </label>
        </div>

        <div class="hr"></div>

        <div class="split">
          <div class="kpi">
            <div class="t">{{ t("steamSpend") }}</div>
            <div class="v mono">{{ money(leg1Spend) }}</div>
          </div>
          <div class="kpi">
            <div class="t">{{ t("csfloatGain") }}</div>
            <div class="v mono">{{ money(leg1Gain) }}</div>
          </div>
        </div>

        <div class="errSlot">
          <div v-if="errors.leg1" class="err">{{ t("steamNotEnough") }}</div>
        </div>
      </div>

      <div class="card">
        <h2>{{ t("leg2Title") }}</h2>

        <div class="form">
          <label>
            {{ t("quantity") }}
            <input type="number" step="1" v-model.number="s.leg2.qty" />
          </label>

          <label>
            {{ t("csfloatBuy") }}
            <input type="number" step="0.01" v-model.number="s.leg2.csfloatBuyPerItem" />
          </label>

          <label>
            {{ t("steamNetReceive") }}
            <input type="number" step="0.01" v-model.number="s.leg2.steamNetPerItem" />
          </label>
        </div>

        <div class="hr"></div>

        <div class="split">
          <div class="kpi">
            <div class="t">{{ t("csfloatSpend") }}</div>
            <div class="v mono">{{ money(leg2Spend) }}</div>
          </div>
          <div class="kpi">
            <div class="t">{{ t("steamGain") }}</div>
            <div class="v mono">{{ money(leg2Gain) }}</div>
          </div>
        </div>

        <div class="errSlot">
          <div v-if="errors.leg2" class="err">{{ t("csfloatNotEnough") }}</div>
        </div>
      </div>
    </div>

    <div class="card" style="margin-top:14px">
      <h2>{{ t("result") }}</h2>

      <div class="split">
        <div class="kpi">
          <div class="t">{{ t("finalSteam") }}</div>
          <div class="v mono">{{ money(afterLeg2.steam) }}</div>
        </div>

        <div class="kpi">
          <div class="t">{{ t("netProfit") }}</div>
          <div class="v mono">{{ money(netProfit) }}</div>
          <div class="small">ROI: {{ pct(roiPct) }}</div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { reactive, computed, watch, onMounted } from "vue";

type Lang = "EN" | "TR";

const lang = reactive<{ current: Lang }>({ current: "EN" });

const toggleLang = () => {
  lang.current = lang.current === "EN" ? "TR" : "EN";
};

const currentLang = computed(() => lang.current);

const dictionary = {
  EN: {
    title: "Steam ↔ CSFloat Arbitrage",
    subtitle: "Balance flow simulation",
    reset: "Reset",
    startingBalances: "Starting Balances",
    initialSteam: "Initial Steam Wallet ($)",
    initialCsfloat: "Initial CSFloat Balance ($)",
    leg1Title: "Leg 1: Steam → CSFloat",
    leg2Title: "Leg 2: CSFloat → Steam",
    quantity: "Quantity",
    steamBuy: "Steam Buy (per item)",
    csfloatNet: "CSFloat Net (per item)",
    csfloatBuy: "CSFloat Buy (per item)",
    steamNetReceive: "Steam Net (per item)",
    steamSpend: "Steam Spend",
    csfloatGain: "CSFloat Gain",
    csfloatSpend: "CSFloat Spend",
    steamGain: "Steam Gain",
    steamNotEnough: "Not enough Steam balance",
    csfloatNotEnough: "Not enough CSFloat balance",
    result: "Result",
    finalSteam: "Final Steam Wallet",
    netProfit: "Net Profit"
  },
  TR: {
    title: "Steam ↔ CSFloat Arbitraj",
    subtitle: "Bakiye akış simülasyonu",
    reset: "Sıfırla",
    startingBalances: "Başlangıç Bakiyeleri",
    initialSteam: "Başlangıç Steam Bakiyesi ($)",
    initialCsfloat: "Başlangıç CSFloat Bakiyesi ($)",
    leg1Title: "Adım 1: Steam → CSFloat",
    leg2Title: "Adım 2: CSFloat → Steam",
    quantity: "Adet",
    steamBuy: "Steam Alış (adet)",
    csfloatNet: "CSFloat Net (adet)",
    csfloatBuy: "CSFloat Alış (adet)",
    steamNetReceive: "Steam Net (adet)",
    steamSpend: "Steam Harcama",
    csfloatGain: "CSFloat Kazanç",
    csfloatSpend: "CSFloat Harcama",
    steamGain: "Steam Kazanç",
    steamNotEnough: "Steam bakiyesi yetersiz",
    csfloatNotEnough: "CSFloat bakiyesi yetersiz",
    result: "Sonuç",
    finalSteam: "Final Steam Bakiyesi",
    netProfit: "Net Kâr"
  }
};

const t = (key: keyof typeof dictionary.EN) =>
  dictionary[lang.current][key];

const s = reactive({
  initialSteam: 0,
  initialCsfloat: 0,
  leg1: { qty: 0, steamBuyPerItem: 0, csfloatNetPerItem: 0 },
  leg2: { qty: 0, csfloatBuyPerItem: 0, steamNetPerItem: 0 }
});

const q = (v: number) => Math.max(0, Math.floor(v));
const n = (v: number) => Number(v) || 0;

const leg1Spend = computed(() => q(s.leg1.qty) * n(s.leg1.steamBuyPerItem));
const leg1Gain = computed(() => q(s.leg1.qty) * n(s.leg1.csfloatNetPerItem));
const leg2Spend = computed(() => q(s.leg2.qty) * n(s.leg2.csfloatBuyPerItem));
const leg2Gain = computed(() => q(s.leg2.qty) * n(s.leg2.steamNetPerItem));

const afterLeg1 = computed(() => ({
  steam: n(s.initialSteam) - leg1Spend.value,
  csfloat: n(s.initialCsfloat) + leg1Gain.value
}));

const afterLeg2 = computed(() => ({
  steam: afterLeg1.value.steam + leg2Gain.value,
  csfloat: afterLeg1.value.csfloat - leg2Spend.value
}));

const netProfit = computed(() => afterLeg2.value.steam - n(s.initialSteam));
const roiPct = computed(() =>
  n(s.initialSteam) > 0
    ? (netProfit.value / n(s.initialSteam)) * 100
    : 0
);

const errors = computed(() => ({
  leg1: leg1Spend.value > n(s.initialSteam),
  leg2: leg2Spend.value > afterLeg1.value.csfloat
}));

const money = (v: number) =>
  v.toLocaleString("en-US", { style: "currency", currency: "USD" });

const pct = (v: number) => v.toFixed(2) + "%";

const resetAll = () => {
  Object.assign(s, {
    initialSteam: 0,
    initialCsfloat: 0,
    leg1: { qty: 0, steamBuyPerItem: 0, csfloatNetPerItem: 0 },
    leg2: { qty: 0, csfloatBuyPerItem: 0, steamNetPerItem: 0 }
  });
};

watch(lang, () => {
  localStorage.setItem("lang", lang.current);
});

onMounted(() => {
  const savedLang = localStorage.getItem("lang") as Lang;
  if (savedLang) lang.current = savedLang;
});
</script>