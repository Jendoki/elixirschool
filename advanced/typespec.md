%{
  version: "1.0.3",
  title: "Specifications et types",
  excerpt: """
  Dans cette leçon, nous allons voir les syntaxes `@spec` et `@type`.
  `@spec` est plutôt un complément syntaxique pour écrire de la documentation qui pourra être analysée par des outils.
  `@type` nous aide à écrire du code plus facile à lire et à comprendre.
  """
}
---

## Introduction

Il est commun de vouloir décrire l'interface de nos fonctions.
On pourrait utiliser [l'annotation @doc](/en/lessons/basics/documentation), mais c'est uniquement de l'information pour les autres développeurs, qui n'est pas vérifiée au moment de la compilation.
Pour ça, Elixir a l'annotation `@spec`, qui décrit les spécificités d'une fonction, et sera vérifié par le compilateur.

Dans certains cas, la spécification peut être assez compliquée.
Afin de réduire la complexité du code, nous pouvons utiliser les types personnalisés. 
Elixir met en place l'annotation `@type` pour cet usage.
Mais Elixir est un langage dynamique. Ce qui veut dire que les informations du type seront ignorées par le compilateur, mais peuvent être utilisées par d'autres outils.

## Spécification

Si vous avez de l'expérience avec Java, vous pouvez penser aux spécifications comme des `interfaces`.
La spécification définie de quels types devraient être les paramètres de la fonction, et la valeur de retour.

Pour définir le type des entrées et sorties, on utilise la directive `@spec` juste au-dessus de la définition de la fonction. Elle prend en paramètres le nom de la fonction, une liste de types des paramètres, et après `::`, le type de la valeur de retour.

Regardons un exemple :

```elixir
@spec sum_product(integer) :: integer
def sum_product(a) do
  [1, 2, 3]
  |> Enum.map(fn el -> el * a end)
  |> Enum.sum()
end
```

Tout a l'air correct, et quand on l'appelle, un résultat valide sera retourné, mais la fonction `Enum.sum` retourne un `number`, pas un `integer` comme attendu par le `@spec`.
Cela pourrait être une source de bugs! Il y a des outils comme Dialyzer pour faire une analyse statique du code, qui nous aiderait à trouver ce genre de bug.
Nous en parlerons dans une autre leçon.

## Les types personnalisés

Parfois, nos fonctions ont besoin de structures de données plus compliquées que de simples nombres et collections.
Dans ces cas, avec `@spec`, cela pourrait être dur à comprendre et à modifier pour d'autres développeurs.
Les fonctions peuvent avoir besoin d'un grand nombre de paramètres, ou de retourner des données complexes. 
Dans les langages orientés objet comme Ruby ou Java, nous pourrions facilement définir des classes, qui nous aideraient à résoudre ce problème.
Elixir n'a pas de classes, mais parce qu'il est facile de l'étendre, on peut définir nos propres types.

Elixir contient des types simples, comme `integer` ou `pid`.
Vous pouvez trouver la liste de tout les types disponibles dans la [documentation](https://hexdocs.pm/elixir/typespecs.html#types-and-their-syntax).

### Définir des types personnalisés

Nous allons modifier notre fonction `sum_times` et lui donner des paramètres en plus :

```elixir
@spec sum_times(integer, %Examples{first: integer, last: integer}) :: integer
def sum_times(a, params) do
  for i <- params.first..params.last do
    i
  end
  |> Enum.map(fn el -> el * a end)
  |> Enum.sum()
  |> round
end
```

Nous avons créé une struct dans le module `Examples` qui contient deux champs - `first` et `last`.
C'est une version plus simple de la struct du module `Range`.
Pour plus d'information sur les `structs`, vous pouvez voir la section dédiée dans le cours sur les [modules](/en/lessons/basics/modules#structs).
Imaginons que nous ayons besoin d'une spécification avec une struct `Examples` à plusieurs endroits.
Il serait ennuyeux de devoir écrire une spécification longue et complexe, et cela pourrait être une source de bugs.
La solution à ce problème est `@type`.

Elixir a trois directives pour les types :

- `@type` – type simple et public.
La structure interne du type est publique.
- `@typep` – type privé, peut être utilisé seulement dans le module ou il est défini.
- `@opaque` – type public, mais dont la structure interne est privée.

Définissons notre type :

```elixir
defmodule Examples do
  defstruct first: nil, last: nil

  @type t(first, last) :: %Examples{first: first, last: last}

  @type t :: %Examples{first: integer, last: integer}
end
```

Nous avons déjà défini le type `t(first, last)`, qui est une représentation de la struct `%Examples{first: first, last: last}`.
On sait que les types pourraient prendre des paramètres, mais nous avons aussi défini le type `t`, et cette fois, c'est une représentation de la struct `%Examples{first: integer, last: integer}`.

Quelle est la différence ? Le premier représente la struct `Examples` dans laquelle les deux clés pourraient être de n'importe quel type.
La deuxième représente la struct dans laquelle les clés sont des `integers`.
Ce qui veut dire que ce code :

```elixir
@spec sum_times(integer, Examples.t()) :: integer
def sum_times(a, params) do
  for i <- params.first..params.last do
    i
  end
  |> Enum.map(fn el -> el * a end)
  |> Enum.sum()
  |> round
end
```

Est égal à ce code :

```elixir
@spec sum_times(integer, Examples.t(integer, integer)) :: integer
def sum_times(a, params) do
  for i <- params.first..params.last do
    i
  end
  |> Enum.map(fn el -> el * a end)
  |> Enum.sum()
  |> round
end
```

### Documentation de types

Le dernier élément dont nous devons parler est comment documenter nos types.
Comme nous le savons grâce à la leçon [documentation](/en/lessons/basics/documentation), il existe les annotations `@doc` et `@moduledoc` pour créer de la documentation pour les fonctions et modules.
Pour documenter nos types, nous pouvons utiliser `@typedoc` :

```elixir
defmodule Examples do
  @typedoc """
      Type that represents Examples struct with :first as integer and :last as integer.
  """
  @type t :: %Examples{first: integer, last: integer}
end
```

La directive `@typedoc` est similaire à `@doc` et `@moduledoc`.