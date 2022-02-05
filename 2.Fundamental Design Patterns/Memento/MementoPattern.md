# :mortar_board: Memento 패턴

> memento 패턴에 대해서 알아보자!

memento 패턴을 사용하면 오브젝트를 **저장하고 복원** 할 수 있다.

![memento](/2.Fundamental%20Design%20Patterns/Memento/memento.png)

1. **Originator (발신자)** 는 오브젝트를 저장하거나 복원한다.

2. **Memento (저장대상)** 는 저장된 상태를 나타낸다.

3. **Caretaker (관리자)** 는 발신자에게 오브젝트의 저장을 요청하고 저장대상의 응답을 전달 받는다. 관리자는 저장대상을 지속시키는 책임이 있으며, 나중에 memento를 다시 발신자에게 제공하여 발신자의 상태를 복원한다.

iOS 앱은 일반적으로 인코더(Encoder)를 사용하여 발신자의 상태를 저장대상으로 인코딩하고, 디코더(Decoder)를 사용하여 저장대상을 다시 발신자로 디코딩한다. 이렇게 하면 인코딩 및 디코딩 로직을 발신자들 간에 재사용이 가능하다.
  
예를들어 JSONEncoder와 JSONDecoder는 각각 JSON 데이터로 오브젝트를 인코딩 및 디코딩 할수있도록 한다.

> 위 예시에서는 JSON 데이터가 Memento(저장대상)가 된다.

## 언제 사용될 수 있을까?

오브젝트의 상태를 저장하거나 복원하려고 할때마다 memento 패턴을 사용 할 수 있다.
  
예를들어, memento 패턴을 사용하여 게임 저장 시스템을 구현할 수있다. 게임 상태(레벨, 체력, 목숨의 수 등)는 Originator(발신자)이고, memento는 저장된 데이터이다. 그리고 Caretaker(관리자)는 게임 시스템이 된다.

또한 이전 상태들의 스택과 같은 memento들의 배열을 유지할수도 있다. 이것을 이용하여 IDE 또는 그래픽 소프트웨어에서의 되돌리기/다시하기 기능을 구현 할수있다.

> 말그대로 저장대상에 대해서 저장 및 복원이 가능하고 이를 관리하는 관리자가 제어를 하는 형태의 패턴인것 같다!

## 예시 코드

Memento는 패턴 분류에서 보면 행동 패턴으로 분류되어 있다. 왜냐하면 저장 및 복원 행동에 대한 패턴이기 때문이다.

* Originator

```swift
import Foundation

// MARK: - Originator
public class Game: Codable {

    public class State: Codable {
        public var attemptsRemaining: Int = 3
        public var level: Int = 1
        public var score: Int = 0
    }

    public var state = State()

    public func rackUpMassivePoints() {
        state.score += 9002
    }

    public func monstersEatPlayer() {
        state.attemptsRemaining -= 1
    }
}
```

* Memento

```swift
// MARK: - Memento
typealias GameMemento = Data
```

* CareTaker

```swift
// MARK: - CareTaker
public class GameSystem {

    private let decoder = JSONDecoder() // 복원
    private let encoder = JSONEncoder() // 저장
    private let userDefaults = UserDefaults.standard

    public func save(_ game: Game, title: String) throws {
        let data = try encoder.encode(game)
        userDefaults.set(data, forKey: title)
    }

    public func load(title: String) throws -> Game {
        guard let data = userDefulats.data(forKey: title),
        let game = try? decoder.decode(Game.self, from: data)
        else {
            throw Error.gameNotFound
        }
        return game
    }

    public enum Error: String, Swift.Error {
        case gameNotFound
    }
}
```

* Example On Playground
  
```swift
// MARK: - Example
var game = Game()
game.monstersEatplayer()
game.rackUpMassivePoints()

// Save Game
let gameSystem = GameSystem()
try gameSystem.save(game, title: "Best Game Ever")

// New Game
game = Game()
print("New Game Score: \(game.state.score)") // 0

// Load Game
game = try! gameSystem.load(title: "Best Game Ever")
print("Loaded Game Score: \(game.state.score)") // 9002
```

## 핵심 요약

* Memento 패턴은 오브젝트의 저장 및 복원이 가능하다. 발신자(Originator), 저장대상(Memento), 관리자(CareTaker)의 세가지 타입으로 이루어져 있다.

* 발신자는 저장할 오브젝트, 저장 대상은 저장된 상태, 관리자는  연관 저장대상들의 유지 및 제어한다.

* iOS는 저장대상을 인코딩 및 디코딩 하기위한 Encoder, Decoder를 제공한다. 이것은 인코딩 및 디코딩 로직을 여러 발신자에서 사용할수 있다.

***
##### Artwork/images/designs: from Design Patterns by Tutorials, available at www.raywenderlich.com