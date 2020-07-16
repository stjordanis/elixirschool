---
version: 1.4.1
title: Ενότητες
---

Ξέρουμε από πείρας ότι είναι ακατάστατο να έχουμε όλες τις συναρτήσεις μας στο ίδιο αρχείο και με το ίδιο πεδίο δράσης.
Σε αυτό το μάθημα θα καλύψουμε το πως να συλλέγουμε συναρτήσεις και να ορίζουμε ένα ειδικό χάρτη γνωστό ως δομή ώστε να οργανώνουμε πιο αποτελεσματικά τον κώδικά μας.

{% include toc.html %}

## Ενότητες

Οι ενότητες είναι ο καλύτερος τρόπος οργάνωσης συναρτήσεων σε ένα namespace.
Επιπρόσθετα της συλλογής συναρτήσεων, μας επιτρέπουν να ορίζουμε ονομασμένες και ιδιωτικές συναρτήσεις τις οποίες καλύψαμε στο [μάθημα συναρτήσεων](../functions/)

Ας δούμε ένα βασικό παράδειγμα:

``` elixir
defmodule Example do
  def greeting(name) do
    "Γεια σου #{name}."
  end
end

iex> Example.greeting "Sean"
"Γειά σου Sean."
```

Είναι δυνατόν να ενσωματώσουμε ενότητες στην Elixir, επιτρέποντας μας να ομαδοποιήσουμε περαιτέρω την λειτουργικότητά μας:

```elixir
defmodule Example.Greetings do
  def morning(name) do
    "Καλημέρα #{name}."
  end

  def evening(name) do
    "Καληνύχτα #{name}."
  end
end

iex> Example.Greetings.morning "Sean"
"Καλημέρα Sean."
```

### Ορίσματα Ενοτήτων

Τα ορίσματα ενοτήτων είναι πιο συχνά ως σταθερές στην Elixir.
Ας δούμε ένα απλό παράδειγμα:

```elixir
defmodule Example do
  @greeting "Γεια σου"

  def greeting(name) do
    ~s(#{@greeting} #{name}.)
  end
end
```

Είναι σημαντικό να σημειώσουμε ότι υπάρχουν κατειλημμένα ορίσματα στην Elixir.
Τα τρία πιο συχνά είναι:

+ `moduledoc` — Τεκμηριώνει την τρέχουσα ενότητα.
+ `doc` — Τεκμηρίωση συναρτήσεων και μακροεντολών.
+ `behaviour` — Χρήσης μιας OTP ή μιας καθορισμένης από το χρήστη συμπεριφοράς.

## Δομές

Οι δομές είναι ειδικοί χάρτες με ένα ορισμένο σύνολο κλειδιών και προκαθορισμένες τιμές.
Μια δομή πρέπει να είναι ορισμένη μέσα σε μια ενότητα, από την οποία παίρνει και το όνομά της.
Είναι συχνό για μια δομή να είναι το μόνο πράγμα που ορίζεται σε μια ενότητα.

Για να ορίσουμε μια δομή χρησιμοποιούμε την `defstruct` μαζί με μια λίστα λέξεων κλειδί από πεδία και προκαθορισμένες τιμές:

```elixir
defmodule Example.User do
  defstruct name: "Sean", roles: []
end
```

Ας δημιουργήσουμε μερικές δομές:

```elixir
iex> %Example.User{}
%Example.User<name: "Sean", roles: [], ...>

iex> %Example.User{name: "Steve"}
%Example.User<name: "Steve", roles: [], ...>

iex> %Example.User{name: "Steve", roles: [:manager]}
%Example.User<name: "Steve", roles: [:manager]>
```

Μπορούμε να αναβαθμίσουμε μια δομή όπως θα το κάναμε σε ένα χάρτη:

```elixir
iex> steve = %Example.User{name: "Steve"}
%Example.User<name: "Steve", roles: [...], ...>
iex> sean = %{steve | name: "Sean"}
%Example.User<name: "Sean", roles: [...], ...>
```

Σημαντικότερα, μπορούμε να αντιπαραβάλουμε δομές με χάρτες:

```elixir
iex> %{name: "Sean"} = sean
%Example.User<name: "Sean", roles: [...], ...>
```

Από την Elixir 1.8 οι δομές περιλαμβάνουν προσαρμοσμένη προεπισκόπηση.
Για να κατανοήσουμε τι σημαίνει αυτό και πως να το χρησιμοποιήσουμε, ας προεπισκοπήσουμε το χάρτη `sean`:

```elixir
iex> inspect(sean)
"%Example.User<name: \"Sean\", roles: [...], ...>"
```

Όλα τα πεδία μας είναι παρόντα, το οποίο αρκεί για αυτό το παράδειγμα, αλλά τι θα συνέβαινε αν είχαμε ένα προστατευόμενο πεδίο που δεν θέλαμε να συμπεριλάβουμε;
Το νέο χαρακτηριστικό `@derive` μας επιτρέπει να πετύχουμε ακριβώς αυτό!
Ας αναβαθμίσουμε το παράδειγμά μας ώστε η μεταβλητή `roles` δεν περιλαμβάνεται πλέον στην έξοδο:

```elixir
defmodule Example.User do
  @derive {Inspect, only: [:name]}
  defstruct name: nil, roles: []
end
```

_Σημείωση_: θα μπορούσαμε επίσης να χρησιμοποιήσουμε την εντολή `@derive {Inspect, except: [:roles]}`, είναι το ίδιο πράγμα.

Με την αναβαθμισμένη ενότητά μας ας δούμε τι συμβαίνει στο `iex`:

```elixir
iex> sean = %Example.User{name: "Sean"}
%Example.User<name: "Sean", ...>
iex> inspect(sean)
"%Example.User<name: \"Sean\", ...>"
```

Οι ρόλοι δεν συμπεριλαμβάνονται πλέον στην έξοδο!

## Σύνθεση

Τώρα που ξέρουμε πως να δημιουργήσουμε ενότητες και δομές ας μάθουμε πως να προσθέσουμε λειτουργικότητα σε αυτές μέσω της σύνθεσης.
Η Elixir μας παρέχει μια ποικιλία διαφορετικών τρόπων για να αλληλεπιδρούμε με άλλες ενότητες.

### `alias`

Μας επιτρέπει να δίνουμε ψευδώνυμο σε ονόματα ενοτήτων. Χρησιμοποιείται πολύ συχνά σε κώδικα Elixir:

```elixir
defmodule Sayings.Greetings do
  def basic(name), do: "Γεια σου, #{name}"
end

defmodule Example do
  alias Sayings.Greetings

  def greeting(name), do: Greetings.basic(name)
end

# Χωρίς ψευδώνυμο

defmodule Example do
  def greeting(name), do: Sayings.Greetings.basic(name)
end
```

Αν υπάρχει σύγκρουση μεταξύ δύο ψευδωνύμων ή απλά επιθυμούμε να δώσουμε ψευδώνυμο ένα τελείως διαφορετικό όνομα, μπορούμε να χρησιμοποιήσουμε την επιλογή της `:as`:

```elixir
defmodule Example do
  alias Sayings.Greetings, as: Hi

  def print_message(name), do: Hi.basic(name)
end
```

Είναι επίσης εφικτό να δώσουμε ψευδώνυμο σε πολλές ενότητες μαζί:

```elixir
defmodule Example do
  alias Sayings.{Greetings, Farewells}
end
```

### `import`

Αν θέλουμε να εισάγουμε συναρτήσεις και μακροεντολές αντί να δώσουμε ψευδώνυμο στην ενότητα τότε χρησιμοποιούμε την `import`:

```elixir
iex> last([1, 2, 3])
** (CompileError) iex:9: undefined function last/1
iex> import List
nil
iex> last([1, 2, 3])
3
```

#### Φιλτράρισμα

Είναι προκαθορισμένο όλες οι συναρτήσεις και οι μακροεντολές να εισάγονται αλλά μπορούμε να τις φιλτράρουμε χρησιμοποιώντας τις επιλογές `:only` και `:except` .

Για να εισάγουμε συγκεκριμένες συναρτήσεις και μακροεντολές, πρέπει να παρέχουμε το ζευγάρι όνομα/τάξη στις `:only` και `:except`.
Ας ξεκινήσουμε εισάγοντας μόνο την συνάρτηση `last/1`:

```elixir
iex> import List, only: [last: 1]
iex> first([1, 2, 3])
** (CompileError) iex:13: undefined function first/1
iex> last([1, 2, 3])
3
```

Αν εισάγουμε τα πάντα εκτός της `last/1` και δοκιμάσουμε τις ίδιες συναρτήσεις με πριν:

```elixir
iex> import List, except: [last: 1]
nil
iex> first([1, 2, 3])
1
iex> last([1, 2, 3])
** (CompileError) iex:3: undefined function last/1
```

Επιπρόσθετα στα ζεύγη όνομα/τάξη υπάρχουν δύο ειδικά άτομα, τα `:functions` και `:macros`, τα οποία εισάγουν μόνο συναρτήσεις και μακροεντολές αντίστοιχα:

```elixir
import List, only: :functions
import List, only: :macros
```

### `require`

Θα μπορούσαμε να χρησιμοποιήσουμε την `require` για να πούμε στην Elixir ότι θα χρησιμοποιήσουμε macros από άλλες ενότητες.
Η μικρή διαφορά με την `import` είναι ότι επιτρέπει την χρήση μακροεντολών, αλλά όχι συναρτήσεων από την οριζόμενη ενότητα:

```elixir
defmodule Example do
  require SuperMacros

  SuperMacros.do_stuff
end
```

Αν προσπαθήσουμε να καλέσουμε μια μακροεντολη η οποία δεν έχει φορτωθεί ακόμα η Elixir θα σηκώσει ένα σφάλμα.

### `use`

Με τη χρήση της `use` μακροεντολής μπορούμε να επιτρέψουμε σε μια άλλη ενότητα να μετατρέψει τον ορισμό της τρέχουσας ενότητας.
Όταν καλούμε την `use` στον κώδικά μας στην πραγματικότητα τρέχουμε την `__using__/1` επανάκληση που ορίζεται στην παρεχόμενη ενότητα.
Το αποτέλεσμα της μακροεντολής `__using__/1` γίνεται μέρος του ορισμού της ενότητάς μας.
Για να καταλάβετε καλύτερα πως λειτουργεί αυτό ας δούμε ένα απλό παράδειγμα:

```elixir
defmodule Hello do
  defmacro __using__(_opts) do
    quote do
      def hello(name), do: "Hi, #{name}"
    end
  end
end
```

Εδώ δημιουργούμε μια νέα ενότητα `Hello` που ορίζει την επανάκληση `__using__/1` μέσα από την οποία ορίζουμε τη συνάρτηση `hello/1`.
Ας δημιουργήσουμε μια νέα ενότητα ώστε να δοκιμάσουμε το νέο μας κώδικα:

```elixir
defmodule Example do
  use Hello
end
```

Αν δοκιμάσουμε τον κώδικά μας στο IEx θα δούμε ότι η `hello/1` είναι διαθέσιμη στην ενότητα `Example`:

```elixir
iex> Example.hello("Sean")
"Hi, Sean"
```

Εδώ μπορούμε να δούμε ότι η `use` κάλεσε την `__using__/1` επανάκληση στην ενότητα `Hello` η οποία με τη σειρά της πρόσθεσε το αποτέλεσμα σαν κώδικα στην ενότητά μας.
Τώρα που δείξαμε ένα απλό παράδειγμα ας αναβαθμίσουμε τον κώδικά μας ώστε να δούμε πως η `__using__/1` μπορεί να υποστηρίξει επιλογές.
Θα το κάνουμε με την προσθήκη μιας επιλογής `greeting`:

```elixir
defmodule Hello do
  defmacro __using__(opts) do
    greeting = Keyword.get(opts, :greeting, "Hi")

    quote do
      def hello(name), do: unquote(greeting) <> ", " <> name
    end
  end
end
```

Ας αναβαθμίσουμε την ενότητά μας `Example` ώστε να περιλαμβάνει την νέα μας επιλογή `greeting`:

```elixir
defmodule Example do
  use Hello, greeting: "Hola"
end
```

Αν την δοκιμάσουμε στο IEx θα δούμε ότι ο χαιρετισμός άλλαξε:

```elixir
iex> Example.hello("Sean")
"Hola, Sean"
```

Αυτά είναι απλά παραδείγματα για να δείξουμε πως λειτουργεί η `use` αλλά είναι ένα εξαιρετικά δυνατό εργαλείο στην εργαλειοθήκη της Elixir.
Καθώς θα μαθαίνετε περισσότερα για την Elixir έχετε το νού σας στην `use`, ένα παράδειγμα που θα δείτε σίγουρα είναι το `use ExUnit.Case, async: true`.

**Σημείωση**: οι `quote`, `alias`, `use` και `require` είναι μακροεντολές που χρησιμοποιούνται όταν δουλεύουμε με τον [μεταπρογραμματισμό](../../advanced/metaprogramming).
