# イントロダクション

*速度を取るか単純さを取るか? **どちらも**取ってみては？*

`pest`はテキストをパースするためのRustで実装されたライブラリです。

`pest`を使ったパーサは[Parsing Expression Grammars]、もしくは*PEGs*を使用することにより、**簡単にデザインすることができ、メンテナンスすることができます。**
そして、Rustのゼロコスト抽象化により、`pest`で書かれたパーサはとても早く動作します。

## サンプル

ここに書いてあるのは、[このあとの章で作成する（まだ書き途中です）](examples/calculator.html)単純な計算機のgrammerです。

```pest
num = @{ int ~ ("." ~ ASCII_DIGIT*)? ~ (^"e" ~ int)? }
    int = { ("+" | "-")? ~ ASCII_DIGIT+ }

operation = _{ add | subtract | multiply | divide | power }
    add      = { "+" }
    subtract = { "-" }
    multiply = { "*" }
    divide   = { "/" }
    power    = { "^" }

expr = { term ~ (operation ~ term)* }
term = _{ num | "(" ~ expr ~ ")" }

calculation = _{ SOI ~ expr ~ EOI }

WHITESPACE = _{ " " | "\t" }
```

そして次に書かれているのが先程のパーサを利用して実際に計算をする関数です。

```rust
lazy_static! {
    static ref PREC_CLIMBER: PrecClimber<Rule> = {
        use Rule::*;
        use Assoc::*;

        PrecClimber::new(vec![
            Operator::new(add, Left) | Operator::new(subtract, Left),
            Operator::new(multiply, Left) | Operator::new(divide, Left),
            Operator::new(power, Right)
        ])
    };
}

fn eval(expression: Pairs<Rule>) -> f64 {
    PREC_CLIMBER.climb(
        expression,
        |pair: Pair<Rule>| match pair.as_rule() {
            Rule::num => pair.as_str().parse::<f64>().unwrap(),
            Rule::expr => eval(pair.into_inner()),
            _ => unreachable!(),
        },
        |lhs: f64, op: Pair<Rule>, rhs: f64| match op.as_rule() {
            Rule::add      => lhs + rhs,
            Rule::subtract => lhs - rhs,
            Rule::multiply => lhs * rhs,
            Rule::divide   => lhs / rhs,
            Rule::power    => lhs.powf(rhs),
            _ => unreachable!(),
        },
    )
}
```

## この本について

この本は`pest`についての概要といくつかのサンプルを書いています。より詳しく`pest`のAPIについて知りたい場合は[ドキュメント]をご覧ください。

`pest`はRustの複雑な機能を使うので、Rustについて知りたい方は[Rust本]（日本語版）をご覧ください。

[Parsing Expression Grammars]: grammars/peg.html
[ドキュメント]: https://docs.rs/pest/
[Rust本]: https://doc.rust-jp.rs/book/second-edition/
