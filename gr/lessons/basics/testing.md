---
version: 1.2.0
title: Δοκιμές
---

Οι δοκιμές είναι ένα σημαντικό μέρος της ανάπτυξης λογισμικού.
Σε αυτό το μάθημα θα δούμε πως να δοκιμάζουμε των κωδικά μας σε Elixir με το ExUnit και μερικές από τις καλύτερες πρακτικές για να το κάνουμε.

{% include toc.html %}

## ExUnit

Το ενσωματωμένο περιβάλλον δοκιμών της Elixir είναι το ExUnit και περιλαμβάνει ότι χρειαζόμαστε για να δοκιμάσουμε τον κώδικά μας διεξοδικά.
Πριν προχωρήσουμε, είναι σημαντικό να σημειώσουμε ότι οι δοκιμές εφαρμόζονται σαν Elixir scripts, έτσι πρέπει να χρησιμοποιήσουμε την επέκταση αρχείου `.exs`.
Πριν μπορέσουμε να τρέξουμε τις δοκιμές μας πρέπει να ξεκινήσουμε το ExUnit με το `ExUnit.start()`, το οποίο συνήθως γίνεται στο `test/test_helper.exs`.

Όταν δημιουργήσαμε το δοκιμαστικό project μας στο προηγούμενο μάθημα, το mix ήταν αρκετά βολικό στο να μας δημιουργήσει μία απλή δοκιμή, την οποία βρίσκουμε στο `test/example_test.exs`:

```elixir
defmodule ExampleTest do
  use ExUnit.Case
  doctest Example

  test "greets the world" do
    assert Example.hello() == :world
  end
end
```

Μπορούμε να τρέξουμε τις δοκιμές του project μας με την εντολή `mix test`.
Αν το κάνουμε τώρα θα πρέπει να δούμε μια παρόμοια έξοδο:

```shell
..

Finished in 0.03 seconds
1 tests, 0 failures
```

Γιατί υπάρχουν δύο τελείες στην έξοδο των δοκιμών; Εκτός από τη δοκιμή στο αρχείο `test/example_test.exs`, το Mix επίσης δημιούργησε μία δοκιμή τεκμηρίωσης στο αρχείο `lib/example.ex`.

```elixir
defmodule Example do
  @moduledoc """
  Documentation for Example.
  """

  @doc """
  Hello world.

  ## Examples

      iex> Example.hello
      :world

  """
  def hello do
    :world
  end
end
```

### assert

Αν έχετε γράψει δοκιμές στο παρελθόν τότε θα είστε εξοικειομένοι με την `assert`.  σε μερικά περιβάλλοντα θα βρείτε τις `should` ή `expect` να παίρνουν την θέση της.

Χρησιμοποιούμε την μακροεντολή `assert` για να δοκιμάσουμε ότι η έκφραση είναι αληθής.
Στην περίπτωση που δεν είναι, ένα σφάλμα θα σηκωθεί και οι δοκιμές μας θα αποτύχουν.
Για να δοκιμάσουμε μια αποτυχία ας αλλάξουμε το δείγμα μας και τότε να τρέξουμε την `mix test`:

```elixir
defmodule ExampleTest do
  use ExUnit.Case
  doctest Example

  test "greets the world" do
    assert Example.hello() == :word
  end
end
```

Τώρα θα πρέπει να δούμε μια πολύ διαφορετική έξοδο:

```shell
  1) test greets the world (ExampleTest)
     test/example_test.exs:5
     Assertion with == failed
     code:  assert Example.hello() == :word
     left:  :world
     right: :word
     stacktrace:
       test/example_test.exs:6 (test)

.

Finished in 0.03 seconds
2 tests, 1 failures
```

Το ExUnit θα μας πει που ακριβώς οι εκτιμήσεις μας αποτυγχάνουν, ποιά ήταν η προσδοκώμενη τιμή και ποιά ήταν στην πραγματικότητα.

### refute

Η `refute` είναι για την `assert` ότι η `unless` για την `if`.
Χρησιμοποιήστε την `refute` όταν θέλετε να βεβαιωθείτε ότι μια έκφραση είναι πάντα ψευδής.

### assert_raise

Μερικές φορές μπορεί να είναι απαραίτητο να βεβαιωθούμε ότι σηκώθηκε κάποιο σφάλμα.
Μπορούμε να το κάνουμε αυτό με την `assert_raise`.
Θα δούμε ένα παράδειγμα της `assert_raise` στο επόμενο μάθημα για το Plug.

### assert_receive

Στις εφαρμογές Elixir αποτελούνται από ηθοποιούς/διεργασίες που στέλνουν μηνύματα η μία στην άλλη, έτσι θέλετε να δοκιμάζετε τα μηνύματα που στέλνονται.
Από τη στιγμή που η ExUnit τρέχει στη δική της διεργασία, μπορεί να λάβει μηνύματα όπως κάθε άλλη συνάρτηση και μπορείτε να κάνετε εκτιμήσεις πάνω τους με την μακροεντολή `assert_received`:

```elixir
defmodule SendingProcess do
  def run(pid) do
    send(pid, :ping)
  end
end

defmodule TestReceive do
  use ExUnit.Case

  test "receives ping" do
    SendingProcess.run(self())
    assert_received :ping
  end
end
```

Η `assert_received` δεν περιμένει για μηνύματα, με την `assert_receive` μπορείτε να ορίσετε ένα όριο λήξης.

### capture_io και capture_log

Η σύλληψη της εξόδου μιας εφαρμογής είναι δυνατή με την `ExUnit.CaptureIO` χωρίς να αλλάξουμε την αρχική εφαρμογή.
Απλά περάστε την συνάρτηση που παράγει την έξοδο σε αυτήν:

```elixir
defmodule OutputTest do
  use ExUnit.Case
  import ExUnit.CaptureIO

  test "outputs Hello World" do
    assert capture_io(fn -> IO.puts("Hello World") end) == "Hello World\n"
  end
end
```

Η `ExUnit.CaptureLog` είναι η αντίστοιχη ενότητα για τη σύλληψη της εξόδου στην `Logger`.

## Ρυθμίσεις Δοκιμών

Σε μερικές περιπτώσεις μπορεί να είναι απαραίτητο να κάνουμε κάποιες ρυθμίσεις πριν τις δοκιμές μας.
Για να το πετύχουμε αυτό μπορούμε να χρησιμοποιήσουμε τις μακροεντολές `setup` και `setup_all`.
Η `setup` θα τρέξει πριν από κάθε δοκιμή και η `setup_all` μια φορά πριν από την σουίτα.
Αναμένεται να επιστρέψουν μια τούπλα του στυλ `{:ok, state}`, με την κατάσταση να είναι διαθέσιμη για τις δοκιμές μας.

Για χάρη του παραδείγματος, θα αλλάξουμε τον κώδικά μας να χρησιμοποιεί την `setup_all`:

```elixir
defmodule ExampleTest do
  use ExUnit.Case
  doctest Example

  setup_all do
    {:ok, recipient: :world}
  end

  test "greets", state do
    assert Example.hello() == state[:recipient]
  end
end
```

## Μίμηση

Η απλή απάντηση για μίμηση στην Elixir: μην το κάνετε.
Μπορεί ενστικτωδώς να ψάχνετε για μιμήσεις αλλά η κοινότητα της Elixir τις αποθαρρύνει και για καλό λόγο.

Για μια εκτενέστερη συζήτηση, υπάρχει αυτό το [εξαιρετικό άρθρο](http://blog.plataformatec.com.br/2015/10/mocks-and-explicit-contracts/).
Η ουσία είναι ότι αντί να μιμείστε την ύπαρξη εξαρτήσεων για τις δοκιμές σας, υπάρχουν πολλά πλεονεκτήματα στο να ορίσετε διεπαφές (συμπεριφορές) για κώδικα εκτός της εφαρμογής σας και να χρησιμοποιείτε υλοποιήσεις μίμησης στις δοκιμές σας.

Για να αλλάξετε τις υλοποιήσεις στον κώδικα εφαρμογής σας, ο προτιμώμενος τρόπος είναι να περάσετε την ενότητα σαν όρισμα και να χρησιμοποιήσετε μια προκαθορισμένη τιμή.
Αν αυτό δεν λειτουργεί, χρησιμοποιήστε τον υπάρχοντα μηχανισμό ρυθμίσεων.
Για τη δημιουργία αυτών των υλοποιήσεων μίμησης, δεν χρειάζεστε μια ειδική βιβλιοθήκη μίμησης, μόνο συμπεριφορές και επανακλήσεις.
