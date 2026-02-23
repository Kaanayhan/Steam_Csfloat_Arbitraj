<template>
  <div class="wrap">
    <div class="top">
      <div>
        <h1>{{ t("title") }}</h1>
        <p class="sub">{{ t("subtitle") }}</p>
      </div>

      <div class="actions">
        <button @click="toggleLang" title="Language">
          <span class="pill">{{ currentLang }}</span>
          <span class="muted">{{ t("lang") }}</span>
        </button>

        <button class="ghost" @click="fillExample">
          {{ t("example") }}
        </button>

        <button class="primary" @click="resetAll">
          {{ t("reset") }}
        </button>
      </div>
    </div>

    <div class="card cardRow">
      <div class="cardHead">
        <h2>{{ t("startingBalances") }}</h2>
        <span class="badge">{{ t("tipBalances") }}</span>
      </div>

      <div class="form">
        <label>
          {{ t("initialSteam") }}
          <input
            type="number"
            step="0.01"
            inputmode="decimal"
            v-model.number="s.initialSteam"
            :class="{ badInput: errors.leg1 }"
          />
          <span class="help">{{ t("helpSteam") }}</span>
        </label>

        <label>
          {{ t("initialCsfloat") }}
          <input
            type="number"
            step="0.01"
            inputmode="decimal"
            v-model.number="s.initialCsfloat"
            :class="{ badInput: errors.leg2 }"
          />
          <span class="help">{{ t("helpCsfloat") }}</span>
        </label>
      </div>
    </div>

    <div class="grid">
      <div class="card">
        <div class="cardHead">
          <h2>{{ t("leg1Title") }}</h2>
          <span class="badge">{{ t("leg1Badge") }}</span>
        </div>

        <div class="form">
          <label>
            {{ t("quantity") }}
            <input type="number" step="1" inputmode="numeric" v-model.number="s.leg1.qty" />
          </label>

          <label>
            {{ t("steamBuy") }}
            <input type="number" step="0.01" inputmode="decimal" v-model.number="s.leg1.steamBuyPerItem" />
          </label>

          <label class="span2">
            {{ t("csfloatNet") }}
            <input type="number" step="0.01" inputmode="decimal" v-model.number="s.leg1.csfloatNetPerItem" />
            <span class="help">{{ t("helpNet") }}</span>
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
        <div class="cardHead">
          <h2>{{ t("leg2Title") }}</h2>
          <span class="badge">{{ t("leg2Badge") }}</span>
        </div>

        <div class="form">
          <label>
            {{ t("quantity") }}
            <input type="number" step="1" inputmode="numeric" v-model.number="s.leg2.qty" />
          </label>

          <label>
            {{ t("csfloatBuy") }}
            <input type="number" step="0.01" inputmode="decimal" v-model.number="s.leg2.csfloatBuyPerItem" />
          </label>

          <label class="span2">
            {{ t("steamNetReceive") }}
            <input type="number" step="0.01" inputmode="decimal" v-model.number="s.leg2.steamNetPerItem" />
            <span class="help">{{ t("helpNet") }}</span>
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
      <div class="cardHead">
        <h2>{{ t("result") }}</h2>
        <span
          class="status"
          :class="profitState"
          :title="profitState"
        >
          <span class="dot" :class="profitState"></span>
          <span class="mono">{{ money(netProfit) }}</span>
          <span class="muted">• ROI {{ pct(roiPct) }}</span>
        </span>
      </div>

      <div class="split">
        <div class="kpi">
          <div class="t">{{ t("finalSteam") }}</div>
          <div class="v mono">{{ money(afterLeg2.steam) }}</div>
          <div class="small">
            {{ t("afterLeg2Explain") }}
          </div>
        </div>

        <div class="kpi">
          <div class="t">{{ t("finalCsfloat") }}</div>
          <div class="v mono">{{ money(afterLeg2.csfloat) }}</div>
          <div class="small">
            {{ t("afterLeg1Explain") }}
          </div>
        </div>
      </div>

      <div class="hr"></div>

      <div class="mini">
        <div class="row">
          <span class="muted">{{ t("leg1Summary") }}</span>
          <span class="mono">{{ money(leg1Spend) }} → {{ money(leg1Gain) }}</span>
        </div>
        <div class="row">
          <span class="muted">{{ t("leg2Summary") }}</span>
          <span class="mono">{{ money(leg2Spend) }} → {{ money(leg2Gain) }}</span>
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
    subtitle: "Simple balance flow simulator",
    lang: "Lang",
    reset: "Reset",
    example: "Fill Example",
    startingBalances: "Starting Balances",
    tipBalances: "Set what you have now",
    initialSteam: "Steam Wallet ($)",
    initialCsfloat: "CSFloat Balance ($)",
    helpSteam: "Your available Steam wallet balance.",
    helpCsfloat: "Your available CSFloat balance.",
    leg1Title: "Step 1: Steam → CSFloat",
    leg2Title: "Step 2: CSFloat → Steam",
    leg1Badge: "Buy on Steam, sell on CSFloat",
    leg2Badge: "Buy on CSFloat, sell on Steam",
    quantity: "Quantity",
    steamBuy: "Steam Buy (per item)",
    csfloatNet: "CSFloat Net Receive (per item)",
    csfloatBuy: "CSFloat Buy (per item)",
    steamNetReceive: "Steam Net Receive (per item)",
    helpNet: "Net means after fees.",
    steamSpend: "Steam Spend",
    csfloatGain: "CSFloat Gain",
    csfloatSpend: "CSFloat Spend",
    steamGain: "Steam Gain",
    steamNotEnough: "Not enough Steam balance",
    csfloatNotEnough: "Not enough CSFloat balance",
    result: "Result",
    finalSteam: "Final Steam Wallet",
    finalCsfloat: "Final CSFloat Balance",
    afterLeg2Explain: "Steam after step 1 + step 2 net receive",
    afterLeg1Explain: "CSFloat after step 1 - step 2 spend",
    leg1Summary: "Step 1 totals",
    leg2Summary: "Step 2 totals"
  },
  TR: {
    title: "Steam ↔ CSFloat Arbitraj",
    subtitle: "Basit bakiye akış simülatörü",
    lang: "Dil",
    reset: "Sıfırla",
    example: "Örnek Doldur",
    startingBalances: "Başlangıç Bakiyeleri",
    tipBalances: "Şu anki bakiyeni gir",
    initialSteam: "Steam Cüzdanı ($)",
    initialCsfloat: "CSFloat Bakiyesi ($)",
    helpSteam: "Mevcut Steam cüzdan bakiyen.",
    helpCsfloat: "Mevcut CSFloat bakiyen.",
    leg1Title: "Adım 1: Steam → CSFloat",
    leg2Title: "Adım 2: CSFloat → Steam",
    leg1Badge: "Steam’den al, CSFloat’ta sat",
    leg2Badge: "CSFloat’tan al, Steam’de sat",
    quantity: "Adet",
    steamBuy: "Steam Alış (adet)",
    csfloatNet: "CSFloat Net Geçen (adet)",
    csfloatBuy: "CSFloat Alış (adet)",
    steamNetReceive: "Steam Net Geçen (adet)",
    helpNet: "Net = kesintiler sonrası.",
    steamSpend: "Steam Harcama",
    csfloatGain: "CSFloat Kazanç",
    csfloatSpend: "CSFloat Harcama",
    steamGain: "Steam Kazanç",
    steamNotEnough: "Steam bakiyesi yetersiz",
    csfloatNotEnough: "CSFloat bakiyesi yetersiz",
    result: "Sonuç",
    finalSteam: "Final Steam Bakiyesi",
    finalCsfloat: "Final CSFloat Bakiyesi",
    afterLeg2Explain: "Adım 1 sonrası + Adım 2 net gelen",
    afterLeg1Explain: "Adım 1 sonrası - Adım 2 harcama",
    leg1Summary: "Adım 1 toplam",
    leg2Summary: "Adım 2 toplam"
  }
};

const t = (key: keyof typeof dictionary.EN) => dictionary[lang.current][key];

const s = reactive({
  initialSteam: 0,
  initialCsfloat: 0,
  leg1: { qty: 0, steamBuyPerItem: 0, csfloatNetPerItem: 0 },
  leg2: { qty: 0, csfloatBuyPerItem: 0, steamNetPerItem: 0 }
});

const clampQty = (v: number) => Math.max(0, Math.floor(Number(v) || 0));
const clampMoney = (v: number) => Math.max(0, Number(v) || 0);

const leg1Spend = computed(() => clampQty(s.leg1.qty) * clampMoney(s.leg1.steamBuyPerItem));
const leg1Gain = computed(() => clampQty(s.leg1.qty) * clampMoney(s.leg1.csfloatNetPerItem));
const leg2Spend = computed(() => clampQty(s.leg2.qty) * clampMoney(s.leg2.csfloatBuyPerItem));
const leg2Gain = computed(() => clampQty(s.leg2.qty) * clampMoney(s.leg2.steamNetPerItem));

const afterLeg1 = computed(() => ({
  steam: clampMoney(s.initialSteam) - leg1Spend.value,
  csfloat: clampMoney(s.initialCsfloat) + leg1Gain.value
}));

const afterLeg2 = computed(() => ({
  steam: afterLeg1.value.steam + leg2Gain.value,
  csfloat: afterLeg1.value.csfloat - leg2Spend.value
}));

const netProfit = computed(() => afterLeg2.value.steam - clampMoney(s.initialSteam));
const roiPct = computed(() =>
  clampMoney(s.initialSteam) > 0 ? (netProfit.value / clampMoney(s.initialSteam)) * 100 : 0
);

const errors = computed(() => ({
  leg1: leg1Spend.value > clampMoney(s.initialSteam),
  leg2: leg2Spend.value > afterLeg1.value.csfloat
}));

const profitState = computed(() => {
  if (netProfit.value > 0) return "ok";
  if (netProfit.value < 0) return "bad";
  return "warn";
});

const money = (v: number) => {
  const locale = lang.current === "TR" ? "tr-TR" : "en-US";
  return v.toLocaleString(locale, { style: "currency", currency: "USD" });
};

const pct = (v: number) => v.toFixed(2) + "%";

const resetAll = () => {
  Object.assign(s, {
    initialSteam: 0,
    initialCsfloat: 0,
    leg1: { qty: 0, steamBuyPerItem: 0, csfloatNetPerItem: 0 },
    leg2: { qty: 0, csfloatBuyPerItem: 0, steamNetPerItem: 0 }
  });
};

const fillExample = () => {
  Object.assign(s, {
    initialSteam: 200,
    initialCsfloat: 50,
    leg1: { qty: 2, steamBuyPerItem: 40, csfloatNetPerItem: 37.5 },
    leg2: { qty: 2, csfloatBuyPerItem: 35, steamNetPerItem: 38 }
  });
};

watch(
  () => lang.current,
  () => localStorage.setItem("lang", lang.current)
);

watch(
  s,
  () => {
    // sanitize inputs continuously
    s.initialSteam = clampMoney(s.initialSteam);
    s.initialCsfloat = clampMoney(s.initialCsfloat);
    s.leg1.qty = clampQty(s.leg1.qty);
    s.leg2.qty = clampQty(s.leg2.qty);
    s.leg1.steamBuyPerItem = clampMoney(s.leg1.steamBuyPerItem);
    s.leg1.csfloatNetPerItem = clampMoney(s.leg1.csfloatNetPerItem);
    s.leg2.csfloatBuyPerItem = clampMoney(s.leg2.csfloatBuyPerItem);
    s.leg2.steamNetPerItem = clampMoney(s.leg2.steamNetPerItem);
  },
  { deep: true }
);

onMounted(() => {
  const savedLang = localStorage.getItem("lang") as Lang | null;
  if (savedLang === "EN" || savedLang === "TR") lang.current = savedLang;
});
</script>