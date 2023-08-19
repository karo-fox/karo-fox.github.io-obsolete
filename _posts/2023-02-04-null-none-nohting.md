---
layout: post
title: Null None Nothing
description: Opisuję, w jaki sposób różne języki programowania radzą sobie z wyrażaniem braku wartości
date: 2023-02-04 15:16:18 +0100
category: języki programowania
tags: programowanie
---

Ostatnio brałam udział w dyskusji dotyczącej wyrażania braku wartości w kodzie. Rozmowa dość szybko wygasła, skłoniła mnie jednak do pochylenia się nad tematem.

# **_It's a trap_**

W "Czystym Kodzie" zwracanie i przekazywanie `null` nazywane jest złą praktyką i główną operacją powodującą powstawanie błędów[^1]. Sam pomysłodawca `null reference`, Tony Hoare, mówi o niej _the billion dollar mistake_ [^2].

Żeby uratować się przed `NullPointerException` i podobnymi potworkami w wielu językach na każdym kroku musimy sprawdzać, czy zmienna nie przechowuje wartości `null`. Powoduje to "nieczystość" kodu i utrudnia jego zrozumienie.

Kolejnym problemem jest znaczenie `null`. Przykładowo, co oznacza `null` przypisane do zmiennej przechowującej adres? Czy adres jest nieznany? Czy jeszcze nie został wprowadzony, a spodziewamy się, że będzie? Czy po prostu nie został jeszcze zainicjalizowany? A może `null` oznacza, że adres jest tajny i nie mamy możliwości odczytania go?

Tak więc jasne jest, że `null` może sprawiać problemy. Niewątpliwie jest jednak czymś przydatnym. Na szczęście współcześnie języki programowania udostępniają wiele sposobów na ułatwienie pracy z pustymi wartościami.

# JavaScript i TypeScript

`null` to jedna z `primitive values` języka JavaScript. Reprezentuje _celowy_ brak jakiejkolwiek wartości. Język ten posiada również inną wartość: `undefined`. Ona z kolei oznacza, że obiektowi _jeszcze_ nie została przypisana wartość.

Takie rozróżnienie niweluje jeden ze wspomnianych problemów. Jeśli przykładowy adres jest `undefined`, to oczekuje na swoją wartość. Jeśli zaś pod adresem kryje się `null`, to taki brak wartości jest celowy.

Dodatkowe możliwości oferuje korzystanie z TypeScripta. Ustawienie opcji `TSConfig strictNullChecks` na `true` sprawia, że `undefined` i `null` otrzymują własne typy. Jeśli będą one używane w miejscu, gdzie oczekiwana jest konkretna wartość, zostanie wywołany `TypeError`.

# Dart, C#, Kotlin

To języki, które oferują różne operatory ułatwiające pracę z wartościami `null`. Są proste i wygodne w użyciu oraz nie utrudniają zrozumienia kodu.

Zmienne w tych językach są domyślnie non-nullable, co oznacza, że nie mogą przechowywać wartości `null` bez naszego "pozwolenia". Oznaczyć zmienną jako nullable możemy przez użycie dodanie `?` do typu zmiennej przy jej deklaracji, np. w Darcie:

```dart
String? name
```

Równie przydatny jest operator `??` w Darcie i C#, `?:` w Kotlinie. Jego zastosowanie wygląda tak, na przykładzie Darta:

```dart
String playerName = name ?? "Guest";
```

Jeśli wyrażenie po lewej stronie nie jest nullem, operator je zwróci. W przeciwnym wypadku operator zwróci wyrażenie po prawej stronie.

Języki te oferują jeszcze więcej metod na null-safety. Co ważne, są one bardzo wygodne w użyciu i mało problematyczne. Sprawiają, że pilnowanie pustych wartości staje się proste, a pojawienie się nulla w nieoczekiwanym momencie wywoła compile-time error. Dzięki temu mamy szansę na naprawienie buga przed jego wystąpieniem.

# Rust i Haskell

Te języki nie mają odpowiednika `null`.

Rust posiada enumerator `Option<T>`, którego implementacja wygląda tak:

```rs
pub enum Option<T> {
    None,
    Some(T),
}
```

`Option` reprezentuje opcjonalną wartość. Option może mieć wariant `Some`, wtedy przechowuje wartość, lub `None`, kiedy jej nie przechowuje.

Użycie typu `Option` wymusza obsłużenie obu wariantów, gdy chcemy uzyskać dostęp do zmiennej. Tak więc jeśli gdzieś pojawi się brak wartości, to musi on zostać poprawnie obsłużony w kodzie. W ten sposób program zostaje zabezpieczony przed niespodziewanym wystąpieniem nulla.

Podobne rozwiązanie prezentuje Haskell ze swoją monadą `Maybe`, z wariantami `Just` i `Nothing`.

# Podsumowanie

`null` jest problematyczny, ale współcześnie istnieje wiele narzędzi pomagających w rozwiązaniu kłopotów z nim związanych. Zaprezentowałam te, które znam, ale to na pewno nie wszystkie. Chętnie poszerzę swoją wiedzę, więc zachęcam do dzielenia się nią w komentarzach!

---

[^1]: R. C. Martin, _Czysty Kod. Podręcznik dobrego programisty_, Helion 2014, s. 130-132
[^2]: [https://www.infoq.com/presentations/Null-References-The-Billion-Dollar-Mistake-Tony-Hoare/](https://www.infoq.com/presentations/Null-References-The-Billion-Dollar-Mistake-Tony-Hoare/)

Powyższy post został opublikowany oryginalnie 25.05.2022 na platforme OhMyDev
