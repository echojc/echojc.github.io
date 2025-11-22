---
title: dynamically creating tests with ScalaTest
date: 2013-05-12
---

At the Code Retreat run at [Movio](https://movio.co) this weekend we watched
[Corey Haines do the Roman Numerals kata in
Ruby](http://www.youtube.com/watch?v=vX-Yym7166Y). An interesting thing he did
was to list all conversion in a hash and iterate over it to dynamically create
the test for each conversion:

```ruby
describe "Converting arabic numbers to roman numerals" do
  {
    1 => "I",
    2 => "II",
    5 => "V"
    # ...
  }.each_pair do |arabic, roman|
    it "converts #{arabic} to #{roman}" do
      expect(convert(arabic)).to eq(roman)
    end
  end
end
```

Later while attempting to create [Conway’s Game of
Life](http://en.wikipedia.org/wiki/Conway's_Game_of_Life) using TDD I came
across some tests that were repetitive in a similar manner:

```scala
describe ("alive cells") {
  val cell = Cell(Alive)
  it ("should become Dead when there are 0 live neighbours") {
    cell.next(0) should be (Cell(Dead))
  }
  it ("should become Dead when there are 1 live neighbours") {
    cell.next(1) should be (Cell(Dead))
  }
  it ("should become Alive when there are 2 live neighbours") {
    cell.next(2) should be (Cell(Alive))
  }
  it ("should become Alive when there are 3 live neighbours") {
    cell.next(3) should be (Cell(Alive))
  }
  // ...
}
```

It turns out that ScalaTest also supports creating tests using the same style:

```scala
describe ("alive cells") {
  val cell = Cell(Alive)

  Seq(
    (0, Dead),
    (1, Dead),
    (2, Alive),
    (3, Alive)
    // ...
  ) foreach { case (count, state) =>
    it (s"should become $state when there are $count live neighbours") {
      cell.next(count) should be (Cell(state))
    }
  }
}
```

It might take some getting used to, but I think it’s a rather nice way to run
different inputs for the same test. It lets you isolate each input into its own
test case without any code duplication and concisely lists all your test cases
and expected outputs together.

You can see how I did Corey’s Roman Numerals kata in Scala (along with other
katas I have done/will do) [on my GitHub](https://github.com/echojc/scala-kata).
