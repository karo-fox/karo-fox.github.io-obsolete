---
layout: post
title: "Po co rozdzielamy funkcje?"
date: 2023-02-06 13:00:00 +0100
category: czysty kod
tags: programowanie
---

Ostatnio obejrzałam na youtubie film [Object-Oriented Programming is Bad](https://www.youtube.com/watch?v=QM1iUe6IofM). Jedną z porad, których udziela Brian Will, jest: _Don't be afraid of long functions_. A ja wszędzie w trakcie mojej nauki programowania słyszę, że rozdzielanie funkcji jest dobre i absolutnie trzeba to robić. Jasne, sam film w tytule zakłada krytykę niektórych zasad OOP, ale rozdzielanie funkcji nie dotyczy jedynie OOP. Dlatego zdecydowałam się pochylić nad tematem, trochę się pozastanawiać i coś z tego kontrastu opinii wyciągnąć. Czego rezultat tutaj prezentuję.

> Ważna uwaga: Nie mam komercyjnego doświadczenia. Nadal się uczę, mam za sobą kilka umiarkowanie skomplikowanych projektów i zaledwie jedną kontrybucję do projektu open source. Także w żadnym wypadku nie wypowiadam się z pozycji autorytetu, a jedynie przedstawiam swoje przemyślenia.
> {: .prompt-info }

# O co chodzi z rozdzielaniem funkcji?

Każdy, kto zacznie się uczyć programowania, w pewnym momencie usłyszy, że funkcje należy rozdzielać na mniejsze. Jeśli zrobimy to dobrze, otrzymamy konretne korzyści:

- kod będzie znacznie bardziej czytelny
- nie będziemy powtarzać kodu
- nie będziemy mieszać różnych poziomów abstrakcji

Zdecydowanie tych korzyści może być więcej, ale te są dla mnie, jako początkującej, najbardziej istotne.

## Przykład

Oto kod, który napisałam około półtora roku temu:

```py
@action(methods=['post'], detail=True)
def revise(self, request, *args, **kwargs):
    topic = Topic.objects.filter(pk=kwargs['pk'])[0]
    field = Field.objects.filter(pk=kwargs['field_pk'])[0]
    self.set_last_reviewed_today([topic, field])
    revision = self.create_revision(field)
    data = TopicSerializer(topic).data
    return Response(data)

def set_last_reviewed_today(self, objects):
    for o in objects:
        o.last_reviewed = datetime.date.today()
        o.save()

def create_revision(self, field):
    date = self.get_revision_date(field)
    revision = Revision.objects.create(date=date, field=field)
    revision.save()
    return revision

def get_revision_date(self, field):
    interval = datetime.timedelta(days=field.review_frequency)
    date = datetime.date.today() + interval
    return date
```

`revise` to metoda, która definiuje nowy endpoint dla API napisanego przy użyciu django-rest-framework. Co tu można zauważyć? Funkcja ma tylko kilka linii, ale i tak można ją rozdzielić. `Topic.objects.filter(pk=kwargs['pk'])[0]` oraz podobne wyrażenie w kolejnej linii aż kłują w oczy. Wystarczą dwie krótkie funkcje, za którymi ukryjemy warstwę abstrakcji:

```py
def __get_topic(self, pk):
    return Topic.objects.filter(pk=pk)[0]

def __get_field(self, pk):
    return Field.objects.filter(pk=pk)[0]
```

Dzięki ich zastosowaniu kod będzie znacznie bardziej czytelny:

```py
@action(methods=['post'], detail=True)
def revise(self, request, *args, **kwargs):
    topic = self.__get_topic(kwargs['pk'])
    field = self.__get_field(kwargs['field_pk'])
    self.set_last_reviewed_today([topic, field])
    revision = self.create_revision(field)
    data = TopicSerializer(topic).data
    return Response(data)
```

Oczywiście kod ten bez wątpienia wymaga dalszej refaktoryzacji. Również bez wątpienia rozdzielenie go na mniejsze funkcje pomaga w jego zrozumieniu. Zatem o co chodzi Brianowi Willowi?

# _Don't be afraid of long functions_

W swoim materiale Brian Will posługuje się przykładem podobnym do poniższego:

Umówmy się, że mamy funkcję, która wykonuje pewną spójną sekwencję wyrażeń. Kod ten nie powtarza się nigdzie indziej w całym programie. Sama funkcja ma kilkanaście linijek.

Czy dobrym pomysłem jest rozbicie jej na mniejsze funkcje, na przykład tak?:

```py
def func():
    doSomething()
    doMoreThings()
    andMore()
    lastThingToDo()
```

Co na tym zyskujemy? Spójny kod został rozrzucony na cztery różne funkcje. Myślę, że może to wręcz utrudnić jego zrozumienie. Kod i tak już wcześniej się w programie nie powtarzał. Poza tym, jeśli to spójna sekwencja wyrażeń, to i tak korzystanie z niej zawsze będzie zakładać wywołanie dokładnie tych czterech funkcji w tej samej kolejności.

> Świadomie nie wspominam tu o rozdzieleniu warstw abstrakcji, bo też Brian Will w swoim przykładzie o tym nie mówi.
> {: .prompt-info }

# Wnioski

Efekt moich rozmyślań to dość prosty wniosek: rozdzielanie funkcji jest kolejnym (tylko i aż) narzędziem w przyborniku programisty. Jest użyteczne, przydatne i ma ogromne spektrum zastosowań. Jednak myślę, że nie powinno się z niego korzystać _automatycznie_. Należy to robić świadomie.

Dlatego też tak ważne są inne zasady dotyczące czystego kodu. Pamiętajmy o YAGNI (_You Aren't Gonna Need It_). Przecież może się okazać, że z tych małych funkcji nie skorzystamy poza tym jednym przypadkiem. _Nie będą nam potrzebne_. Za to po napisaniu kodu możemy go rozwinąć w przyszłości.

---

Jak zawsze będę, zachęcam do pozostawienia feedbacku, zwrócenia uwagi na moje pomyłki i zdrowej, konstruktywnej krytyki. Ten post, tak samo jak cały blog, powstał jako moje narzędzie do nauki, więc na pewno spodziewam się popełniania przeze mnie błędów.
