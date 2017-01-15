# What is the "Functional Programming"?


### Jaewon Choi

---

# What is the Functional Programming?

---

# Functional programming?

* 함수형 프로그래밍의 대전제
	* Pure function 들로만
	* 다시 말해 Side effect 없는 함수들로 프로그램을 구성한다.
* Side effect가 뭐여?
	* 결과를 돌려주는 이외의 어떤 일
		* 변수 수정
		* 콘솔에 출력하거나 사용자의 입력을 읽어들인다.
		* 예외를 던지거나 오류를 내면서 실행을 중단한다.
	* 아니, 저게 없는데 프로그램을 어떻게 짬?

---

# 가능합니다

* FP는 방식에 대한 제약일 뿐, 표현가능한 프로그램의 종류에 대한 제약이 아님 => Turing complete!
* 게다가 FP를 따르면 대단히 이로움
  => test, 재사용, 병렬화, 일반화, 분석 쉬움
  => 버그가 생길 여지가 훨씬 적음
  => 돈을 잘 벌게 되고, 나라가 부강해짐

---

# Side effect가 있는 프로그램 (Impure function)

```scala
class Cafe {
  
  def buyCoffee(cc: CreditCard): Coffee = {
    
    val cup = new Coffee()
    
    cc.charge(cup.price) // side effect. 신용카드를 청구함
    
    cup
    
  }
}
```

---

# 왜 `cc.charge(cup.price)`는 안좋은가?

* 외부 세계와의 일정한 상호작용이 관여
	* 신용카드 회사와 접촉하여, 거래를 승인하고, 대금을 청구하고...
* 이 함수 자체는 cup을 return 해 주는 것만 해줄 뿐, 그외 모든 동작은 부수적으로 일어남.

* 이러한 side effect 때문에 testing이 어려움
	* 테스트를 위해 실제 카드 이용 대금을 청구할 것인가?

---

# 좀 더 나아진 코드

```scala
class Cafe {
  def buyCoffee(cc: CreditCard, p: Payments): Coffee = {
    val cup = new Coffee()
    p.charge(cc, cup.price)
    cup
  }
}
```

* 아직도 side effect가 있지만 testability가 좀 더 나아졌다.
	* 하지만 test를 위해선 Payments를 mock-up 해야 하고, 그러기 위해선 interface를 만들어야 하고... 귀찮
* 재사용이 안된다. 어떻게 바꾸면 좋을까?

---

# 함수적 해법: Side effect의 제거

* buyCoffee()로 커피 뿐만 아니라 청구건을 하나의 값으로 돌려주게 하자!
```scala
class Cafe {
  def buyCoffee(cc: CreditCard): (Coffee, Charge) = {
    val cup = new Coffee()
    (cup, Charge(cc, cup.price))
  }
}
```
* 정말로 buyCoffee가 하는 일은 값을 돌려 주는 일 밖에 없다.

---

# Charge는 뭥미?

```scala
case class Charge(cc: CreditCard, amount: Double) {

  def combine(other: Charge): Charge =

    if (cc == other.cc)
      Charge(cc, amount + other.amount)
    else
      throw new Exception("Can't combine charges to different cards")
}
```

---

# buyCoffee를 확장해보자!

```scala
class Cafe {
  def buyCoffee(cc: CreditCard): (Coffee, Charge) = ...
  
  def buyCoffees(cc: CreditCard, n: Int): (List[Coffee], Charge) = {
    val purchases: List[(Coffee, Charge)] = List.fill(n)(buyCoffee(cc))
    val (coffees, charges) = purchases.unzip
    (coffees, charges.reduce((c1, c2) => c1.combine(c2))
  }
}
```

* testing이 쉬워졌다!
* 실제 구동 시에는 Payment class가 필요하겠지만, test 할 때는 아오안!
* Charge가 first-class value가 되면서, 햄버거든 피자든 다 팔 수 있게 된다!
	* 즉, bussiness logic 를 좀 더 쉽게 조립할 수 있게 된다!

---

# 청구건을 취합하는 것도 간단하다!

```scala
def coalesce(charges: List[Charge]): List[Charge] =
  charges.groupBy(_.cc).values.map(_.reduce(_ combines _)).toList
```

* 이러한 처음에 어려운 한줄짜리 코드를 one-liner 라고 한다.
*  `_`는 anaonymous function을 위한 구문

---

# What is the (Pure) function?

---

# Review on FP

* FP: pure function으로 프로그래밍하는 것
* Pure function: side effect가 없는 function
* 구체적으로 그래서 side effect와 순수성이라는게 뭐냐?
* 구체적으로 그래서 FP라는게 뭐냐?

---

# the (Pure) function

```scala
A => B
```
* Input: 형식이 A인 모든 값 a
* Output: 형식이 B인 모든 값 b
* a와 b를 연관(associate) 시키되 b가 **오직** a의 값에 의해서만 결정된다는 조건을 만족하는 연산
* 즉, 내부 또는 외부의 상태 변경은 f(a)를 수행하는데 어떠한 영향도 주지 않는다.
	* f(x) = x^2 

---

# the (Pure) function

```scala
A => B
```
* 함수는 주어진 입력으로 뭔가를 계산하는 것 외에 프로그램의 실행에 **어떤 관찰 가능한 영향도 미치지 않는다**.
=> Side effect가 없다
=> 이러한 함수를 Pure function이라고 부른다.
=> 별말 없으면 이 책에서 모든 function은 pure function이다.

---

# the SIMPLEST pure function

```scala
3 + 3
```
* `+`는 가장 간단한 pure function
* 이 함수는 같은 입력에 대해서는 항상 같은 값을 돌려준다.

---

# Referential transparency

* Expression의 속성 중 하나.
* Expression은 뭐임?

---

# Expression

* 프로그램을 구성하는 코드 중 하나의 결과로 평가될 수 있는 임의의 코드 조각
* `3 + 5 + 4` 이런거

--- 

# 다시, Referential transparency

* 임의의 프로그램에서 만일 어떤 expression을 evaluated value로 바꾸어도 프로그램의 의미가 변하지 않는다면, 그 expression은 RT하다.
* 만일 expression f(x)가 RT인 모든 x에 대해서 RT하면, 함수 f는 pure.
* 참 쉽죠?

---

# RT, Purity, and Substitution model

---

# 다시 예제로

```scala
def buyCoffee(cc: CreditCard): Coffee = {
  val cup = new Coffee()
  cc.charge(cup.price)
  cup
}
```
* buyCoffee가 순수해 지려면?
* 임의의 p에 대해 p(buyCoffee(aliceCreditCard)) == p(new Coffee())여야 한다.
* Is it?

---

# 다시 RT로

* RT는 함수가 수행하는 모든 것이 함수가 돌려주는 값으로 대표된다는 invariant condiation을 강제한다.
* 이러한 제약을 지키면 substitution model을 적용할 수 있게 된다!

---

# Substitution model

* algebraic equation 푸는 것과 똑같다.
	1. 표현식의 모든 부분을 전개
	2. 모든 변수를 해당 값으로 치환
	3. 그 후 그것을 가장 간단한 형태로 환원(reduce)
* 이것을 equational reasoning 이라고 부른다.
* 즉 substitution model은 equational reasoning 을 통해 evaluation 하는 방법!

---

# RT인 예

```scala
val x = "Hello, Master Seo!"
val r1 = x.reverse // !oeS retsaM ,olleH"
val r2 = x.reverse // same as r1!

// applying SM
val r1 = "Hello, Master Seo!".reverse // same as above!
val r2 = "Hello, Master Seo!".reverse
```
* reverse라는 변환은 결과에 영향을 미치지 않는다.
  * r1과 r2 값이 동일하므로 x는 RT

---

# non-RT 인 예

```scala
val x = new StringBuiler("Hello")
val y = x.append(", Master Seo")
val r1 = y.toString
val r2 = y.toString
```
오키. 여기까진 좋다.

---

# non-RT 인 예

* 모든 y를 x.append(", Master Seo")로 바꿔보자.
```scala
val x = new StringBuiler("Hello")
val r1 = x.append(", Master Seo").toString
val r2 = x.append(", Master Seo").toString

// res:
// Hello, Master Seo, Master Seo
```
* 즉, StringBuiler.append는 pure function이 아니다.

---

# 교훈

* 이렇 듯, side effect가 있으면 결과에 대한 추론이 어려워진다.
  * 전체를 다 알아야 한다.
* 하지만 SM과 함께라면 추론은 매우 쉽다!
  * local 하게만 봐도 해당 함수의 내용을 추론하기 쉽다!
* 좋지 아니한가
  * modulization의 이상 실현: 재사용 가능한 component로 나누면 유지보수도 쉽고, 얘넬 합성(composition)해서 원하는 결과를 내기 좋을거야!

---

# 그런데 아까...

```markdown
* 게다가 FP를 따르면 대단히 이로움
  => test, 재사용, 병렬화, 일반화, 분석 쉬움
  => 버그가 생길 여지가 훨씬 적음
  => **돈을 잘 벌게 되고, 나라가 부강해짐** <= 이거 어디감?
```

---

# Where's FP job market?

* Facebook uses functional programming to make News Feeds run smoothly
http://www.pcworld.com/article/2451480/facebook-uses-functional-programming-to-make-news-feeds-run-smoothly.html
* Haskell opportunities at Facebook
https://www.reddit.com/r/haskell/comments/2useoq/haskell_opportunities_at_facebook/
* 2016's top programming trends
"Increased reliance on functional programming languages"
https://techcrunch.com/2016/12/26/2016s-top-programming-trends/

---

# Every language want to be a FT featured language!

* Java
Do more with less: Lambda expressions in Java 8
http://www.infoworld.com/article/2607502/java/do-more-with-less--lambda-expressions-in-java-8.html

* C#, Python, Javascript, even C++!
	* Work hard C, man.

---

# Many useful frameworks with FT

* Apache Spark
* Akka with Actor model
* Neo4j is replacing their internal implementation to Scala!
* Github's scala projects growing!

---

# Do FP :)

### Jaewon Choi
jaewon.james.choi@gmail.com