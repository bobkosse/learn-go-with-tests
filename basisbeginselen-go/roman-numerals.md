# Inleiding tot property based testen

[**Je kunt hier alle code van dit hoofdstuk vinden**](https://github.com/quii/learn-go-with-tests/tree/main/roman-numerals)

Sommige bedrijven vragen je om de [Romeinse cijferkata](http://codingdojo.org/kata/RomanNumerals/) te doen als onderdeel van het sollicitatiegesprek. Dit hoofdstuk laat zien hoe je dit met TDD kunt aanpakken.

We gaan een functie schrijven die een [Arabisch getal](https://en.wikipedia.org/wiki/Arabic_numerals) (de cijfers 0 tot en met 9) omzet naar een Romeins cijfer.

Als je nog nooit van [Romeinse cijfers](https://en.wikipedia.org/wiki/Roman_numerals) hebt gehoord: dit is de manier hoe de Romeinen vroeger getallen schreven.

Je bouwt ze door symbolen aan elkaar te plakken en die symbolen stellen getallen voor

Zo staat `I` voor "éen" en staat `III` voor drie.

Lijkt makkelijk, maar er zijn een paar interessante regels. `V` betekent vijf, maar `IV` is 4 (niet `IIII`).

`MCMLXXXIV` is 1984. Dat ziet er ingewikkeld uit en het is moeilijk voor te stellen hoe we code kunnen schrijven om dit vanaf het begin uit te zoeken.

Zoals dit boek benadrukt, is een belangrijke vaardigheid voor softwareontwikkelaars het identificeren van "dunne verticale segmenten" van _nuttige_ functionaliteit en vervolgens te **itereren** naar resultaat. De TDD-workflow vergemakkelijkt iteratieve ontwikkeling.

Dus, in plaats van te starten met 1984, laten we beginnen met 1.

## Schrijf eerst de test

```go
func TestRomanNumerals(t *testing.T) {
	got := ConvertToRoman(1)
	want := "I"

	if got != want {
		t.Errorf("got %q, want %q", got, want)
	}
}
```

Als je tot hier in het boek bent gekomen, vind je het hopelijk erg saai en routineus. Dat is goed.

## Probeer de test uit te voeren

```console
./numeral_test.go:6:9: undefined: ConvertToRoman
```

Laat de compiler je de weg wijzen.

## Schrijf de minimale hoeveelheid code om de test te laten uitvoeren en de falende test output te controleren

Creëer onze functie, maar zorg ervoor dat de test nog niet slaagt. Zorg er altijd voor dat de tests falen zoals je verwacht.

```go
func ConvertToRoman(arabic int) string {
	return ""
}
```

De test zou nu moeten werken

```console
=== RUN   TestRomanNumerals
--- FAIL: TestRomanNumerals (0.00s)
    numeral_test.go:10: got '', want 'I'
FAIL
```

## Schrijf genoeg code om de test te laten slagen

```go
func ConvertToRoman(arabic int) string {
	return "I"
}
```

## Refactor

Er is nog niet zoveel te refactoren op dit moment.

_Ik weet_ dat het vreemd voelt om het resultaat gewoon hard te coderen, maar met TDD willen we zo lang mogelijk uit de "rode" situatie blijven. Het voelt misschien alsof we niet veel hebben bereikt, maar we hebben onze API gedefinieerd en een test uitgevoerd die een van onze regels vastlegt, ook al is de "echte" code behoorlijk dom.

Gebruik dat ongemakkelijke gevoel nu om een ​​nieuwe test te schrijven die ons dwingt om iets minder domme code te schrijven.

## Schrijf eerst je test

We kunnen subtests gebruiken om onze tests netjes te groeperen

```go
func TestRomanNumerals(t *testing.T) {
	t.Run("1 gets converted to I", func(t *testing.T) {
		got := ConvertToRoman(1)
		want := "I"

		if got != want {
			t.Errorf("got %q, want %q", got, want)
		}
	})

	t.Run("2 gets converted to II", func(t *testing.T) {
		got := ConvertToRoman(2)
		want := "II"

		if got != want {
			t.Errorf("got %q, want %q", got, want)
		}
	})
}
```

## Probeer de test uit te voeren

```console
=== RUN   TestRomanNumerals/2_gets_converted_to_II
    --- FAIL: TestRomanNumerals/2_gets_converted_to_II (0.00s)
        numeral_test.go:20: got 'I', want 'II'
```

Niet heel veel verbazends hier

## Schrijf genoeg code om de test te laten slagen

```go
func ConvertToRoman(arabic int) string {
	if arabic == 2 {
		return "II"
	}
	return "I"
}
```

Ja, het voelt nog steeds alsof we het probleem niet echt aanpakken. Dus moeten we meer tests schrijven om vooruit te komen.

## Refactor

We hebben wat herhaling in onze tests. Wanneer je iets test waarvan je het gevoel hebt dat het een kwestie is van "gegeven input X, verwachten we Y", kun je beter tabelgebaseerde tests gebruiken.

```go
func TestRomanNumerals(t *testing.T) {
	cases := []struct {
		Description string
		Arabic      int
		Want        string
	}{
		{"1 gets converted to I", 1, "I"},
		{"2 gets converted to II", 2, "II"},
	}

	for _, test := range cases {
		t.Run(test.Description, func(t *testing.T) {
			got := ConvertToRoman(test.Arabic)
			if got != test.Want {
				t.Errorf("got %q, want %q", got, test.Want)
			}
		})
	}
}
```

We kunnen nu eenvoudig meer cases toevoegen zonder dat we nieuwe testboilerplates hoeven te schrijven.

Laten even doorzetten en verder gaan met 3

## Schrijf eerst je test

Voeg het onderstaande toe aan je testcases:

```
{"3 gets converted to III", 3, "III"},
```

## Probeer de test uit te voeren

```console
=== RUN   TestRomanNumerals/3_gets_converted_to_III
    --- FAIL: TestRomanNumerals/3_gets_converted_to_III (0.00s)
        numeral_test.go:20: got 'I', want 'III'
```

## Schrijf genoeg code om de test te laten slagen

```go
func ConvertToRoman(arabic int) string {
	if arabic == 3 {
		return "III"
	}
	if arabic == 2 {
		return "II"
	}
	return "I"
}
```

## Refactor

Oké, ik begin deze if-statements niet meer zo leuk te vinden. En als je goed naar de code kijkt, zie je dat we een string van `I`'s bouwen die gebaseerd is op de waarde van `arabic`.

We "weten" dat we voor ingewikkeldere getallen een soort rekenkunde en het samenvoegen van tekenreeksen zullen moeten toepassen.

Laten we met deze gedachten een refactoring proberen. Het is _misschien niet_ geschikt voor de uiteindelijke oplossing, maar dat is oké. We kunnen onze code altijd weggooien en opnieuw beginnen met de tests die we als leidraad hebben.

```go
func ConvertToRoman(arabic int) string {

	var result strings.Builder

	for i := 0; i < arabic; i++ {
		result.WriteString("I")
	}

	return result.String()
}
```

Je herinnert je misschien [`strings.Builder`](https://golang.org/pkg/strings/#Builder) nog van onze discussie over [benchmarking](iteration.md#benchmarking)

> Een Builder wordt gebruikt om efficiënt een string te bouwen met behulp van Write-methoden. Dit minimaliseert het kopiëren van geheugen.

Normaal gesproken zou ik pas met dit soort optimalisaties beginnen als ik daadwerkelijk een prestatieprobleem heb. Maar de hoeveelheid code is niet veel groter dan een "handmatige" toevoeging aan een string, dus we kunnen net zo goed de snellere aanpak gebruiken.

De code ziet er wat mij betreft beter uit en beschrijft het domein _zoals wij dat nu kennen_.

### De Romeinen kende de DRY-principes ook...

Het wordt nu ingewikkelder. De Romeinen dachten in hun wijsheid dat herhalende tekens moeilijk te lezen en te tellen zouden worden. Een regel met Romeinse cijfers is dus dat je hetzelfde teken niet vaker dan drie keer achter elkaar mag herhalen.

In plaats daarvan neem je het op één na hoogste symbool en verminder je het vervolgens door er een symbool links van te plaatsen. Niet alle symbolen kunnen voor vermindering worden gebruikt; alleen I (1), X (10) en C (100).

Bijvoorbeeld, `5` in Romeinse cijfers is een `V`. Om 4 te maken, doe je niet `IIII`, maar `IV`.

## Schrijf eerst je test

```
{"4 gets converted to IV (can't repeat more than 3 times)", 4, "IV"},
```

## Probeer de test uit te voeren

```console
=== RUN   TestRomanNumerals/4_gets_converted_to_IV_(cant_repeat_more_than_3_times)
    --- FAIL: TestRomanNumerals/4_gets_converted_to_IV_(cant_repeat_more_than_3_times) (0.00s)
        numeral_test.go:24: got 'IIII', want 'IV'
```

## Schrijf genoeg code om de test te laten slagen

```go
func ConvertToRoman(arabic int) string {

	if arabic == 4 {
		return "IV"
	}

	var result strings.Builder

	for i := 0; i < arabic; i++ {
		result.WriteString("I")
	}

	return result.String()
}
```

## Refactor

Ik vind het niet prettig dat we het patroon van het opbouwen van strings hebben doorbroken, en ik zou graag op die manier willen doorgaan.

```go
func ConvertToRoman(arabic int) string {

	var result strings.Builder

	for i := arabic; i > 0; i-- {
		if i == 4 {
			result.WriteString("IV")
			break
		}
		result.WriteString("I")
	}

	return result.String()
}
```

Om 4 te laten "passen" bij mijn huidige denkwijze, tel ik nu af vanaf het Arabische getal en voeg ik naarmate we vorderen symbolen toe aan de reeks. Ik weet niet zeker of dit op de lange termijn zal werken, maar we zullen zien!

Laten we zorgen dat 5 ook werkt&#x20;

## Schrijf eerst de test

```
{"5 gets converted to V", 5, "V"},
```

## Probeer de test uit te voeren

```console
=== RUN   TestRomanNumerals/5_gets_converted_to_V
    --- FAIL: TestRomanNumerals/5_gets_converted_to_V (0.00s)
        numeral_test.go:25: got 'IIV', want 'V'
```

## Schrijf genoeg code om de test te laten slagen

Kopieer gewoon de aanpak die we voor 4 hebben gebruikt

```go
func ConvertToRoman(arabic int) string {

	var result strings.Builder

	for i := arabic; i > 0; i-- {
		if i == 5 {
			result.WriteString("V")
			break
		}
		if i == 4 {
			result.WriteString("IV")
			break
		}
		result.WriteString("I")
	}

	return result.String()
}
```

## Refactor

Herhaling in lussen zoals deze is meestal een teken dat een abstractie wacht om benoemd te worden. Kortsluitende lussen kunnen een effectief hulpmiddel zijn voor leesbaarheid, maar ze kunnen je ook iets anders vertellen.

We gaan over ons Arabische getal heen en als we bepaalde symbolen tegenkomen roepen we `break` aan, maar wat we _eigenlijk_ doen is op een onhandige `i` verlagen.

```go
func ConvertToRoman(arabic int) string {

	var result strings.Builder

	for arabic > 0 {
		switch {
		case arabic > 4:
			result.WriteString("V")
			arabic -= 5
		case arabic > 3:
			result.WriteString("IV")
			arabic -= 4
		default:
			result.WriteString("I")
			arabic--
		}
	}

	return result.String()
}
```

* Als ik goed kijk naar de code, en onze tests van een aantal zeer eenvoudige scenario's, kan ik zien dat ik, om een ​​Romeins cijfer te maken, iets van `arabic` moet aftrekken terwijl ik symbolen toepas.
* De `for`-lus is niet langer afhankelijk van een `i` en in plaats daarvan blijven we de string opbouwen totdat we genoeg waarden van `arabic` hebben afgetrokken.

Ik ben er vrij zeker van dat deze aanpak ook voor 6 (VI), 7 (VII) en 8 (VIII) zal gelden. Voeg de cases desalniettemin toe aan onze testsuite en controleer dit (ik zal de code niet opnemen om dit hoofdstuk niet te lang te maken. Kijk op GitHub voor voorbeelden als je het niet zeker weet hoe je dit doet).

9 volgt dezelfde regel als 4 in die zin dat we 1 moeten aftrekken van de representatie van het volgende getal. 10 wordt in Romeinse cijfers weergegeven met `X`; dus 9 zou `IX` moeten zijn.

## Write the test first

```
{"9 gets converted to IX", 9, "IX"},
```

## Try to run the test

```console
=== RUN   TestRomanNumerals/9_gets_converted_to_IX
    --- FAIL: TestRomanNumerals/9_gets_converted_to_IX (0.00s)
        numeral_test.go:29: got 'VIV', want 'IX'
```

## Write enough code to make it pass

We should be able to adopt the same approach as before

```
case arabic > 8:
    result.WriteString("IX")
    arabic -= 9
```

## Refactor

It _feels_ like the code is still telling us there's a refactor somewhere but it's not totally obvious to me, so let's keep going.

I'll skip the code for this too, but add to your test cases a test for `10` which should be `X` and make it pass before reading on.

Here are a few tests I added as I'm confident up to 39 our code should work

```
{"10 gets converted to X", 10, "X"},
{"14 gets converted to XIV", 14, "XIV"},
{"18 gets converted to XVIII", 18, "XVIII"},
{"20 gets converted to XX", 20, "XX"},
{"39 gets converted to XXXIX", 39, "XXXIX"},
```

If you've ever done OO programming, you'll know that you should view `switch` statements with a bit of suspicion. Usually you are capturing a concept or data inside some imperative code when in fact it could be captured in a class structure instead.

Go isn't strictly OO but that doesn't mean we ignore the lessons OO offers entirely (as much as some would like to tell you).

Our switch statement is describing some truths about Roman Numerals along with behaviour.

We can refactor this by decoupling the data from the behaviour.

```go
type RomanNumeral struct {
	Value  int
	Symbol string
}

var allRomanNumerals = []RomanNumeral{
	{10, "X"},
	{9, "IX"},
	{5, "V"},
	{4, "IV"},
	{1, "I"},
}

func ConvertToRoman(arabic int) string {

	var result strings.Builder

	for _, numeral := range allRomanNumerals {
		for arabic >= numeral.Value {
			result.WriteString(numeral.Symbol)
			arabic -= numeral.Value
		}
	}

	return result.String()
}
```

This feels much better. We've declared some rules around the numerals as data rather than hidden in an algorithm and we can see how we just work through the Arabic number, trying to add symbols to our result if they fit.

Does this abstraction work for bigger numbers? Extend the test suite so it works for the Roman number for 50 which is `L`.

Here are some test cases, try and make them pass.

```
{"40 gets converted to XL", 40, "XL"},
{"47 gets converted to XLVII", 47, "XLVII"},
{"49 gets converted to XLIX", 49, "XLIX"},
{"50 gets converted to L", 50, "L"},
```

Need help? You can see what symbols to add in [this gist](https://gist.github.com/pamelafox/6c7b948213ba55332d86efd0f0b037de).

## And the rest!

Here are the remaining symbols

| Arabic | Roman |
| ------ | :---: |
| 100    |   C   |
| 500    |   D   |
| 1000   |   M   |

Take the same approach for the remaining symbols, it should just be a matter of adding data to both the tests and our array of symbols.

Does your code work for `1984`: `MCMLXXXIV` ?

Here is my final test suite

```go
func TestRomanNumerals(t *testing.T) {
	cases := []struct {
		Arabic int
		Roman  string
	}{
		{Arabic: 1, Roman: "I"},
		{Arabic: 2, Roman: "II"},
		{Arabic: 3, Roman: "III"},
		{Arabic: 4, Roman: "IV"},
		{Arabic: 5, Roman: "V"},
		{Arabic: 6, Roman: "VI"},
		{Arabic: 7, Roman: "VII"},
		{Arabic: 8, Roman: "VIII"},
		{Arabic: 9, Roman: "IX"},
		{Arabic: 10, Roman: "X"},
		{Arabic: 14, Roman: "XIV"},
		{Arabic: 18, Roman: "XVIII"},
		{Arabic: 20, Roman: "XX"},
		{Arabic: 39, Roman: "XXXIX"},
		{Arabic: 40, Roman: "XL"},
		{Arabic: 47, Roman: "XLVII"},
		{Arabic: 49, Roman: "XLIX"},
		{Arabic: 50, Roman: "L"},
		{Arabic: 100, Roman: "C"},
		{Arabic: 90, Roman: "XC"},
		{Arabic: 400, Roman: "CD"},
		{Arabic: 500, Roman: "D"},
		{Arabic: 900, Roman: "CM"},
		{Arabic: 1000, Roman: "M"},
		{Arabic: 1984, Roman: "MCMLXXXIV"},
		{Arabic: 3999, Roman: "MMMCMXCIX"},
		{Arabic: 2014, Roman: "MMXIV"},
		{Arabic: 1006, Roman: "MVI"},
		{Arabic: 798, Roman: "DCCXCVIII"},
	}
	for _, test := range cases {
		t.Run(fmt.Sprintf("%d gets converted to %q", test.Arabic, test.Roman), func(t *testing.T) {
			got := ConvertToRoman(test.Arabic)
			if got != test.Roman {
				t.Errorf("got %q, want %q", got, test.Roman)
			}
		})
	}
}
```

* I removed `description` as I felt the _data_ described enough of the information.
* I added a few other edge cases I found just to give me a little more confidence. With table based tests this is very cheap to do.

I didn't change the algorithm, all I had to do was update the `allRomanNumerals` array.

```go
var allRomanNumerals = []RomanNumeral{
	{1000, "M"},
	{900, "CM"},
	{500, "D"},
	{400, "CD"},
	{100, "C"},
	{90, "XC"},
	{50, "L"},
	{40, "XL"},
	{10, "X"},
	{9, "IX"},
	{5, "V"},
	{4, "IV"},
	{1, "I"},
}
```

## Parsing Roman Numerals

We're not done yet. Next we're going to write a function that converts _from_ a Roman Numeral to an `int`

## Write the test first

We can re-use our test cases here with a little refactoring

Move the `cases` variable outside of the test as a package variable in a `var` block.

```go
func TestConvertingToArabic(t *testing.T) {
	for _, test := range cases[:1] {
		t.Run(fmt.Sprintf("%q gets converted to %d", test.Roman, test.Arabic), func(t *testing.T) {
			got := ConvertToArabic(test.Roman)
			if got != test.Arabic {
				t.Errorf("got %d, want %d", got, test.Arabic)
			}
		})
	}
}
```

Notice I am using the slice functionality to just run one of the tests for now (`cases[:1]`) as trying to make all of those tests pass all at once is too big a leap

## Try to run the test

```console
./numeral_test.go:60:11: undefined: ConvertToArabic
```

## Write the minimal amount of code for the test to run and check the failing test output

Add our new function definition

```go
func ConvertToArabic(roman string) int {
	return 0
}
```

The test should now run and fail

```console
--- FAIL: TestConvertingToArabic (0.00s)
    --- FAIL: TestConvertingToArabic/'I'_gets_converted_to_1 (0.00s)
        numeral_test.go:62: got 0, want 1
```

## Write enough code to make it pass

You know what to do

```go
func ConvertToArabic(roman string) int {
	return 1
}
```

Next, change the slice index in our test to move to the next test case (e.g. `cases[:2]`). Make it pass yourself with the dumbest code you can think of, continue writing dumb code (best book ever right?) for the third case too. Here's my dumb code.

```go
func ConvertToArabic(roman string) int {
	if roman == "III" {
		return 3
	}
	if roman == "II" {
		return 2
	}
	return 1
}
```

Through the dumbness of _real code that works_ we can start to see a pattern like before. We need to iterate through the input and build _something_, in this case a total.

```go
func ConvertToArabic(roman string) int {
	total := 0
	for range roman {
		total++
	}
	return total
}
```

## Write the test first

Next we move to `cases[:4]` (`IV`) which now fails because it gets 2 back as that's the length of the string.

## Write enough code to make it pass

```go
// earlier..
var allRomanNumerals = RomanNumerals{
	{1000, "M"},
	{900, "CM"},
	{500, "D"},
	{400, "CD"},
	{100, "C"},
	{90, "XC"},
	{50, "L"},
	{40, "XL"},
	{10, "X"},
	{9, "IX"},
	{5, "V"},
	{4, "IV"},
	{1, "I"},
}

// later..
func ConvertToArabic(roman string) int {
	var arabic = 0

	for _, numeral := range allRomanNumerals {
		for strings.HasPrefix(roman, numeral.Symbol) {
			arabic += numeral.Value
			roman = strings.TrimPrefix(roman, numeral.Symbol)
		}
	}

	return arabic
}
```

It is basically the algorithm of `ConvertToRoman(int)` implemented backwards. Here, we loop over the given roman numeral string:

* We look for roman numeral symbols taken from `allRomanNumerals`, highest to lowest, at the beginning of the string.
* If we find the prefix, we add its value to `arabic` and trim the prefix.

At the end, we return the sum as the arabic number.

The `HasPrefix(s, prefix)` checks whether string `s` starts with `prefix` and `TrimPrefix(s, prefix)` removes the `prefix` from `s`, so we can proceed with the remaining roman numeral symbols. It works with `IV` and all other test cases.

You can implement this as a recursive function, which is more elegant (in my opinion) but might be slower. I'll leave this up to you and some `Benchmark...` tests.

Now that we have our functions to convert an arabic number into a roman numeral and back, we can take our tests a step further:

## An intro to property based tests

There have been a few rules in the domain of Roman Numerals that we have worked with in this chapter

* Can't have more than 3 consecutive symbols
* Only I (1), X (10) and C (100) can be "subtractors"
* Taking the result of `ConvertToRoman(N)` and passing it to `ConvertToArabic` should return us `N`

The tests we have written so far can be described as "example" based tests where we provide _examples_ for the tooling to verify.

What if we could take these rules that we know about our domain and somehow exercise them against our code?

Property based tests help you do this by throwing random data at your code and verifying the rules you describe always hold true. A lot of people think property based tests are mainly about random data but they would be mistaken. The real challenge about property based tests is having a _good_ understanding of your domain so you can write these properties.

Enough words, let's see some code

```go
func TestPropertiesOfConversion(t *testing.T) {
	assertion := func(arabic int) bool {
		roman := ConvertToRoman(arabic)
		fromRoman := ConvertToArabic(roman)
		return fromRoman == arabic
	}

	if err := quick.Check(assertion, nil); err != nil {
		t.Error("failed checks", err)
	}
}
```

### Rationale of property

Our first test will check that if we transform a number into Roman, when we use our other function to convert it back to a number that we get what we originally had.

* Given random number (e.g `4`).
* Call `ConvertToRoman` with random number (should return `IV` if `4`).
* Take the result of above and pass it to `ConvertToArabic`.
* The above should give us our original input (`4`).

This feels like a good test to build us confidence because it should break if there's a bug in either. The only way it could pass is if they have the same kind of bug; which isn't impossible but feels unlikely.

### Technical explanation

We're using the [testing/quick](https://golang.org/pkg/testing/quick/) package from the standard library

Reading from the bottom, we provide `quick.Check` a function that it will run against a number of random inputs, if the function returns `false` it will be seen as failing the check.

Our `assertion` function above takes a random number and runs our functions to test the property.

### Run our test

Try running it; your computer may hang for a while, so kill it when you're bored :)

What's going on? Try adding the following to the assertion code.

```go
assertion := func(arabic int) bool {
   if arabic < 0 || arabic > 3999 {
   	log.Println(arabic)
   	return true
   }
   roman := ConvertToRoman(arabic)
   fromRoman := ConvertToArabic(roman)
   return fromRoman == arabic
}
```

You should see something like this:

```console
=== RUN   TestPropertiesOfConversion
2019/07/09 14:41:27 6849766357708982977
2019/07/09 14:41:27 -7028152357875163913
2019/07/09 14:41:27 -6752532134903680693
2019/07/09 14:41:27 4051793897228170080
2019/07/09 14:41:27 -1111868396280600429
2019/07/09 14:41:27 8851967058300421387
2019/07/09 14:41:27 562755830018219185
```

Just running this very simple property has exposed a flaw in our implementation. We used `int` as our input but:

* You can't do negative numbers with Roman Numerals
* Given our rule of a max of 3 consecutive symbols we can't represent a value greater than 3999 ([well, kinda](https://www.quora.com/Which-is-the-maximum-number-in-Roman-numerals)) and `int` has a much higher maximum value than 3999.

This is great! We've been forced to think more deeply about our domain which is a real strength of property based tests.

Clearly `int` is not a great type. What if we tried something a little more appropriate?

### [`uint16`](https://golang.org/pkg/builtin/#uint16)

Go has types for _unsigned integers_, which means they cannot be negative; so that rules out one class of bug in our code immediately. By adding 16, it means it is a 16 bit integer which can store a max of `65535`, which is still too big but gets us closer to what we need.

Try updating the code to use `uint16` rather than `int`. I updated `assertion` in the test to give a bit more visibility.

```go
assertion := func(arabic uint16) bool {
	if arabic > 3999 {
		return true
	}
	t.Log("testing", arabic)
	roman := ConvertToRoman(arabic)
	fromRoman := ConvertToArabic(roman)
	return fromRoman == arabic
}
```

Notice that now we are logging the input using the `log` method from the testing framework. Make sure you run the `go test` command with the flag `-v` to print the additional output (`go test -v`).

If you run the test they now actually run and you can see what is being tested. You can run multiple times to see our code stands up well to the various values! This gives me a lot of confidence that our code is working how we want.

The default number of runs `quick.Check` performs is 100 but you can change that with a config.

```go
if err := quick.Check(assertion, &quick.Config{
	MaxCount: 1000,
}); err != nil {
	t.Error("failed checks", err)
}
```

### Further work

* Can you write property tests that check the other properties we described?
* Can you think of a way of making it so it's impossible for someone to call our code with a number greater than 3999?
  * You could return an error
  * Or create a new type that cannot represent > 3999
    * What do you think is best?

## Wrapping up

### More TDD practice with iterative development

Did the thought of writing code that converts 1984 into MCMLXXXIV feel intimidating to you at first? It did to me and I've been writing software for quite a long time.

The trick, as always, is to **get started with something simple** and take **small steps**.

At no point in this process did we make any large leaps, do any huge refactorings, or get in a mess.

I can hear someone cynically saying "this is just a kata". I can't argue with that, but I still take this same approach for every project I work on. I never ship a big distributed system in my first step, I find the simplest thing the team could ship (usually a "Hello world" website) and then iterate on small bits of functionality in manageable chunks, just like how we did here.

The skill is knowing _how_ to split work up, and that comes with practice and with some lovely TDD to help you on your way.

### Property based tests

* Built into the standard library
* If you can think of ways to describe your domain rules in code, they are an excellent tool for giving you more confidence
* Force you to think about your domain deeply
* Potentially a nice complement to your test suite

## Postscript

This book is reliant on valuable feedback from the community. [Dave](http://github.com/gypsydave5) is an enormous help in practically every chapter. But he had a real rant about my use of 'Arabic numerals' in this chapter so, in the interests of full disclosure, here's what he said.

> Just going to write up why a value of type `int` isn't really an 'arabic numeral'. This might be me being way too precise so I'll completely understand if you tell me to f off.
>
> A _digit_ is a character used in the representation of numbers - from the Latin for 'finger', as we usually have ten of them. In the Arabic (also called Hindu-Arabic) number system there are ten of them. These Arabic digits are:
>
> ```console
>   0 1 2 3 4 5 6 7 8 9
> ```
>
> A _numeral_ is the representation of a number using a collection of digits. An Arabic numeral is a number represented by Arabic digits in a base 10 positional number system. We say 'positional' because each digit has a different value based upon its position in the numeral. So
>
> ```console
>   1337
> ```
>
> The `1` has a value of one thousand because its the first digit in a four digit numeral.
>
> Roman are built using a reduced number of digits (`I`, `V` etc...) mainly as values to produce the numeral. There's a bit of positional stuff but it's mostly `I` always representing 'one'.
>
> So, given this, is `int` an 'Arabic number'? The idea of a number is not at all tied to its representation - we can see this if we ask ourselves what the correct representation of this number is:
>
> ```console
> 255
> 11111111
> two-hundred and fifty-five
> FF
> 377
> ```
>
> Yes, this is a trick question. They're all correct. They're the representation of the same number in the decimal, binary, English, hexadecimal and octal number systems respectively.
>
> The representation of a number as a numeral is _independent_ of its properties as a number - and we can see this when we look at integer literals in Go:
>
> ```go
> 	0xFF == 255 // true
> ```
>
> And how we can print integers in a format string:
>
> ```go
> n := 255
> fmt.Printf("%b %c %d %o %q %x %X %U", n, n, n, n, n, n, n, n)
> // 11111111 ÿ 255 377 'ÿ' ff FF U+00FF
> ```
>
> We can write the same integer both as a hexadecimal and an Arabic (decimal) numeral.
>
> So when the function signature looks like `ConvertToRoman(arabic int) string` it's making a bit of an assumption about how it's being called. Because sometimes `arabic` will be written as a decimal integer literal
>
> ```go
> 	ConvertToRoman(255)
> ```
>
> But it could just as well be written
>
> ```go
> 	ConvertToRoman(0xFF)
> ```
>
> Really, we're not 'converting' from an Arabic numeral at all, we're 'printing' - representing - an `int` as a Roman numeral - and `int`s are not numerals, Arabic or otherwise; they're just numbers. The `ConvertToRoman` function is more like `strconv.Itoa` in that it's turning an `int` into a `string`.
>
> But every other version of the kata doesn't care about this distinction so :shrug:
