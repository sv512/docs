# OOP en C++

*Document version 1*

Ce document à pour but de résumer quelques notion sur les classes en C++ via des exemples simples. Il n'est donc pas nécessairement complet

## Déclaration

```cpp
// --- card.hpp --- //

#pragma once // <- fait en sort que le fichier ne soit inclus qu'une fois

class Card {}; // <- doit toujours se terminer par ';'

// --- main.cpp --- //

#include "card.hpp"

int main() {
    Card card; // <- construit l'objet
    some_function_that_uses_card(card);
}
```

## Méthode

```cpp
// --- card.hpp --- //

#pragma once

class Card {
public:
    int hp();
};

// --- card.cpp --- //

#include "card.hpp"

int Card::hp() {
    return 42;
}

// --- main.cpp --- //

#include <iostream>
#include "card.hpp"

int main() {
    Card card;
    std::cout << "hp: "
              << card.hp()
              << '\n';
}
```

## Constructeur

```cpp
// --- card.hpp --- //

#pragma once

class Card {
public:
    Card(int hp);

private:
    int m_hp;
};

// --- card.cpp --- //

#include "card.hpp"

// Type de retour du constructeur évident => on ne le précise pas
Card::Card(int hp) : m_hp(hp) {}

// --- main.cpp --- //

#include "card.hpp"

int main() {
    Card card(42); // <- correct

    // Note :
    // Card card; <- ne marche plus
}
```

## Getter / Setter

```cpp
// --- card.hpp --- //

#pragma once

class Card {
public:
    Card(int hp);

    int hp() const; // <- ne modifie pas l'objet Card => const à la fin
    void set_hp(int hp);

private:
    int m_hp;
};

// --- card.cpp --- //

#include "card.hpp"

Card::Card(int hp) : m_hp(hp) {}

int Card::hp() const {
    return m_hp;
}

void Card::set_hp(int hp) {
    m_hp = hp;
}
```

## Destructeur

```cpp
// --- card.hpp --- //

#pragma once

class Card {
public:
    Card();
    Card(Card familiar);

    ~Card(); // <- déclare le destructeur

private:
    Card *m_familiar;
};

// --- card.cpp --- //

Card::Card() : m_familiar(nullptr) {}
Card::Card(Card familiar) : m_familiar(new Card(familiar)) {}

// Type de retour évident
Card::~Card() {
    if (m_familiar != nullptr)
        delete m_familiar;
}
```

## Opérateur

```cpp
// --- card.hpp --- //

#pragma once

class Card {
public:
    Card(int hp);

    Card operator +(const Card &other) const;

    int hp() const;

private:
    int m_hp;
};

// --- card.cpp --- //

#include "card.hpp"

Card::Card(int hp) : m_hp(hp) {}

Card Card::operator +(const Card &other) const {
    return Card(m_hp + other.m_hp); // <- Card avec les HP additionnés
}

int Card::hp() const {
    return m_hp;
}

// --- main.cpp --- //

#include <iostream>
#include "card.hpp"

int main() {
    Card half_life(21);
    Card life = half_life + half_life;

    std::cout << "life = "
              << life.hp() // <- 42
              << '\n';
}
```

## Fonction amie

```cpp
// --- card.hpp --- //

#pragma once

class Card {
public:
    Card(int hp);

    friend int get_card_hp(const Card &); // <- dans la classe

private:
    int m_hp;
};

// --- card.cpp --- //

#include "card.hpp"

Card::Card(int hp) : m_hp(hp) {}

int get_card_hp(const Card &card) {
    return card.m_hp; // <- grâce à `friend` on peut accéder aux membres privés
}
```

## Affichage

```cpp
// --- card.hpp --- //

#pragma once

#include <ostream>

class Card {
public:
    Card(int hp);

    friend std::ostream &operator <<(std::ostream &, const Card &);

private:
    int m_hp;
};

// --- card.cpp --- //

#include "card.hpp"

Card::Card(int hp) : m_hp(hp) {}

std::ostream &operator <<(std::ostream &output, const Card &card) {
    return output << "hp=" << card.m_hp;
    // ^ est la même chose que :
    // output << "hp=" << card.m_hp;
    // return output;
}
```

> Note : `ostream` signifie "<ins>o</ins>utput <ins>stream</ins>"

> Note : `operator <<` retourne le `std::ostream &` plutôt qu'un `void` pour pouvoir enchaîner les appels :

```cpp
std::cout << 'a' << 'b';

// est en vérité :

(std::cout << 'a') // <- retourne une référence à `std::cout`
           << 'b'; // <- réutilise la ref à `std::cout` retournée
```

> Note : le `ostream` est passé et retourné par référence pour pouvoir écrire dedans sans en faire une copie à chaque fois

## Entrée

```cpp
// --- card.hpp --- //

#pragma once

#include <istream>

class Card {
public:
    Card(int hp);

    // On veut écrire dans la carte => pas de const
    friend std::istream &operator >>(std::istream &, Card &);

private:
    int m_hp;
};

// --- card.cpp --- //

#include "card.hpp"

Card::Card(int hp) : m_hp(hp) {}

std::istream &operator >>(std::istream &input, Card &card) {
    int hp;
    input >> hp;

    if (hp < 1) {
        // HP < 1 => entrée invalide
        input.setstate(std::ios::failbit);
    } else {
        // OK
        card.m_hp = hp;
    }

    return input;
}
```

> Note : `istream` signifie "<ins>i</ins>nput <ins>stream</ins>"

> Note : les même remarques que dans `std::ostream` s'appliquent
