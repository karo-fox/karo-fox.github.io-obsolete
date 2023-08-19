---
layout: post
title: "Niestandardowy input w Vue okiem początkującego"
date: 2023-02-04 15:16:18 +0100
category: tutorial
tags: vue
---

Jestem początkującą programistką, interesuję się głównie backendem. Mówi się, że najlepszym sposobem nauki jest wykonanie projektu. Tak więc od miesiąca w wolnym czasie rozwijam aplikację webową, prosty szyfrator. Jego zadaniem jest przetworzenie przekazanego mu tekstu i zwrócenie zaszyfrowanej wiadomości. Używa prostych szyfrów: Cezara, zamiany... Mam w planach dodać jeszcze kilka. Jednak w międzyczasie naszła mnie ochota na wykonanie frontu do tego projektu. Wybrałam Vue, przeszłam kilka poradników, i strona startowa zaczęła wyglądać całkiem znośnie:

![Encryptor - strona główna](/assets/img/niestandardowy-input-w-vue-okiem-poczatkujacego/niestandartdowy-input-w-vue-okiem-poczatkujacego-img1.png)

Udało mi się wszystko ładnie połączyć, strona prawidłowo komunikuje z API, wszystko działa jak powinno. Ale... Brakuje dość ważnej rzeczy: danych na temat szyfru.

O co chodzi?

### Jak działają używane przeze mnie szyfry?

Szyfr Cezara jest dość popularny. Polega na utworzeniu nowego alfabetu poprzez przesunięcie ciągu znków o kilka pozycji i zamianie każdej litery w szyfrowanej wiadomości na odpowiadającą jej nową literę.

Przykładowo, tak wygląda klucz do szyfrowania utworzony po przesunięciu o trzy pozycje:

![Szyfr Cezara](/assets/img/niestandardowy-input-w-vue-okiem-poczatkujacego/niestandartdowy-input-w-vue-okiem-poczatkujacego-img2.png)

W tym przypadku zaszyfrowana wiadomość "Oh My Dev" to: **Le Jv Abs**

Zamiana to kolejny zaimplementowany przeze mnie szyfr. Jest nieco mniej znany, ale jego działanie jest równie proste. Wymaga klucza, który jest zestawem par niepowtarzających się liter, np. _ga-de-ry-po-lu-ki_. Szyfrowanie polega na zamianie każdej litery w wiadomości na literę, z którą występuje w parze. Jeśli litera nie znajduje się w kluczu, jest pozostawiana bez zmian.

Zaszyfrowana w ten sposób nazwa "Oh My Dev" (klucz: _ga-de-ry-po-lu-ki_) wygląda tak: **Ph Mr Edv**

## W czym tu problem?

Moja strona radzi sobie z przesłaniem wiadomości, a nawet rodzajem szyfru do użycia. Te dane nie wystarczą jednak do przeprowadzenia operacji. Szyfr Cezara wymaga liczby oznaczającej przesunięcie względem alfabetu. Zamiana nie zadziała bez klucza w postaci par liter. Tak więc front mojej aplikacji musi dostarczyć sposobu na przekazanie tych informacji. W tym momencie napotkałam problem: każdy z szyfrów wymaga innego zestawu parametrów, a korzystają one z różnego rodzaju danych. Tak więc strona powinna generować różne pola w zależności od rodzaju wybranego szyfru.

## Zabrałam się więc do roboty

W pędzie kodowania zaczęłam wymyślać i od razu wprowadzać w życie różne pomysły, po chwili je zmieniając i odrzucając coraz to dziwniejsze rozwiązania. Zmęczona nieustającym niepowodzeniem odłożyłam sprawę na później i wyłączyłam komputer. Po kilku godzinach zajmowania się innymi sprawami usiadłam z powrotem do problemu. Tym razem przed kartką i z długopisem w ręce, nawet nie włączając komputera. Po rozpisaniu wszystkiego przed sobą rozwiązanie przyszło od razu: potrzebuję komponentu zawierającego input, ale zdolnego do określania jego typu. Zapewne każdy bardziej doświadczony programista wpadłby na coś takiego bez zastanowienia :D

W każdym razie następnego dnia usiadłam do kodu z nowymi siłami i zaczęłam rozwiązywać mój problem.

## Moje rozwiązanie

_Używam Vue 3 z Composition API, TypeScript_

### Po pierwsze: generowanie pola input

Dzięki przekazaniu danych z komponentu nadrzędnego można w prosty sposób określić parametry elementu input:

```vue
<!-- CustomInput.vue -->
<script setup lang="ts">
import { capitalize, computed } from "vue";

const props = defineProps<{
  inputName: string;
  type: string;
}>();

const label = computed(() => capitalize(props.inputName) + ":");
</script>

<template>
  <label :for="`${inputName}-id`">{{ label }}</label
  ><input :type="type" :id="`${inputName}-id`" :name="inputName" />
</template>
```

### Po drugie: przekazywanie danych z inputu

Vue oferuje dyrektytwę `v-model`, która odpowiada za wykonanie ciężkiej pracy związanej z połączeniem wartości jakiejś zmiennej z zawartością pola input. Działanie `v-model` bardzo dobrze opisuje dokumentacja Vue w tych miejscach:

[https://vuejs.org/guide/essentials/forms.html](https://vuejs.org/guide/essentials/forms.html)

[https://vuejs.org/guide/components/events.html#usage-with-v-model](https://vuejs.org/guide/components/events.html#usage-with-v-model)

Kierując się wskazówkami z dokumentacji dopisałam kilka kolejnych linijek:

```vue
<!-- CustomInput.vue -->
<script setup lang="ts">
import { capitalize, computed } from "vue";

const props = defineProps<{
  inputName: string;
  type: string;
  modelValue: any;
}>();

const emit = defineEmits(["update:modelValue"]);

const label = computed(() => capitalize(props.inputName) + ":");
</script>

<template>
  <label :for="`${inputName}-id`">{{ label }}</label
  ><input
    :type="type"
    :id="`${inputName}-id`"
    :name="inputName"
    :value="modelValue"
    @input="
      $emit('update:modelValue', ($event.target as HTMLInputElement).value)
    "
  />
</template>
```

Teraz całość już spokojnie działa. Komponentu można używać z różnymi rodzajami danych.

Element `input` posiada parametr `value` związany z wartością `modelValue`. Przy zmianie zawartości pola emitowany jest event `update:modelValue`. Dzięki temu możemy używać `v-model` z naszym komponentem:

```vue
<!-- script -->

let value = ref(0);

<!-- template -->
<CustomInput type="number" input-name="transformation" v-model="value" />
```

Vue umożliwia również implementację funkcjonalności `v-model` za pomocą `computed`. W moim przypadku wygląda to tak:

```vue
<!-- CustomInput.vue -->
<script setup lang="ts">
import { capitalize, computed } from "vue";

const props = defineProps<{
  inputName: string;
  type: string;
  modelValue: any;
}>();

const emit = defineEmits(["update:modelValue"]);
let label = computed(() => capitalize(props.inputName) + ":");

const value = computed({
  get() {
    return props.modelValue;
  },
  set(value) {
    emit("update:modelValue", value);
  },
});
</script>

<template>
  <label :for="`${inputName}-id`">{{ label }}</label
  ><input
    :type="type"
    :id="`${inputName}-id`"
    :name="inputName"
    v-model="value"
  />
</template>
```

### Walidacja danych

Dobrze by było, gdyby komponent mógł sprawdzać dane wprowadzane przez użytkownika i wyświetlać pomocne komunikaty. Moje API zadziała poprawnie tylko jeśli jako parametr do szyfru Cezara zostanie podana liczba. Zaś "zamiana" oczekuje ciągu liter o parzystej długości. Dodając `watcher` obserwujący wartość pola input można sprawdzać poprawność otrzymanych informacji i w prosty sposób zdecydować, jaki komunikat wyświetlić. Oto przykład sprawdzający, czy pole nie jest puste:

```vue
<!-- CustomInput.vue -->
<script setup lang="ts">
import { capitalize, computed, ref, watch } from "vue";

const props = defineProps<{
  inputName: string;
  type: string;
  modelValue: any;
}>();

const emit = defineEmits(["update:modelValue"]);
const errorMsg = ref("");
const label = computed(() => capitalize(props.inputName) + ":");

const value = computed({
  get() {
    return props.modelValue;
  },
  set(value) {
    emit("update:modelValue", value);
  },
});

watch(value, (newValue, oldValue) => {
  if (newValue == "") {
    errorMsg.value = "cannot be empty";
  } else {
    errorMsg.value = "";
  }
});
</script>

<template>
  <label :for="`${inputName}-id`">{{ label }}</label
  ><input
    :type="type"
    :id="`${inputName}-id`"
    :name="inputName"
    v-model="value"
  />
  <div>
    {{ errorMsg }}
  </div>
</template>
```

Możemy pójść o krok dalej i przekazywać funkcję `validate` z komponentu nadrzędnego:

```vue
<!-- CustomInput.vue -->
<script setup lang="ts">
import { capitalize, computed, ref, watch } from "vue";

const props = defineProps<{
  inputName: string;
  type: string;
  modelValue: any;
  validate?: Function;
}>();

const emit = defineEmits(["update:modelValue"]);
const errorMsg = ref("");
const label = computed(() => capitalize(props.inputName) + ":");

const value = computed({
  get() {
    return props.modelValue;
  },
  set(value) {
    emit("update:modelValue", value);
  },
});

watch(value, (newValue) => {
  errorMsg.value = props.validate ? props.validate(newValue) : "";
});
</script>

<template>
  <label :for="`${inputName}-id`">{{ label }}</label
  ><input
    :type="type"
    :id="`${inputName}-id`"
    :name="inputName"
    v-model="value"
  />
  <div>
    {{ errorMsg }}
  </div>
</template>

<!-- Parent Component -->
<script setup lang="ts">
import { ref } from "vue";
import CustomInput from "./CustomInput.vue";

const value = ref(0);

function validateNotZero(inputValue: number) {
  return inputValue == 0 ? "cannot be 0" : "";
}
</script>

<template>
  <CustomInput
    type="number"
    input-name="transformation"
    v-model="value"
    :validate="validateNotZero"
  />
</template>
```

## Podsumowanie

W taki sposób stworzyłam komponent, który rozwiązuje mój problem. Po dodaniu stylowania tak prezentuje się efekt końcowy:

![Encryptor - Gotowy niestandardowy input](/assets/img/niestandardowy-input-w-vue-okiem-poczatkujacego/niestandartdowy-input-w-vue-okiem-poczatkujacego-img3.png)

Mój komponent wykorzystam w projekcie dodając do niego walidację dla każdego typu danych, jakich będę porzebować. Mam możliwość dodania kilku pól input i wybrania odpowiedniego typu dla każdego z nich.

Jako że jestem osobą początkującą, będę ogromnie wdzięczna wszystkim, którzy poinformują mnie o błędach w moim kodzie :) Jeśli macie jakieś pomysły jak inaczej można podejść do problemu lub co można usprawnić, podzielcie się swoimi przemyślaniami!

Ostateczny snippet z kodem jest zaprezentowany powyżej, można go również znaleźć na moim Githubie:

[https://github.com/karo-fox/vue-custom-input](https://github.com/karo-fox/vue-custom-input)

Jeśli chcesz sprawdzić moje postępy w projekcie, kod można znaleźć tutaj:

[https://github.com/karo-fox/encryptor](https://github.com/karo-fox/encryptor)

---

Powyższy post został opublikowany oryginalnie 20.03.2022 na platforme OhMyDev
