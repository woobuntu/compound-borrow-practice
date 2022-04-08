[https://medium.com/compound-finance/borrowing-assets-from-compound-quick-start-guide-f5e69af4b8f4](https://medium.com/compound-finance/borrowing-assets-from-compound-quick-start-guide-f5e69af4b8f4)

컴파운드 프로토콜은 사용자로 하여금 다른 가상자산을 담보로 가상자산을 대출할 수 있도록 지원한다.
이는 사용자로 하여금 실제로 소유하지 않은 가상자산으로 거래를 하거나, 애플리케이션을 사용할 수 있도록 하는 등 유연성을 제공하는 것이다.

ex) ETH를 가지고 있는 사용자가 ETH를 컴파운드에 담보로 제공하고 그 대가로 DAI를 대출

컴파운드 프로토콜에 가상자산을 예치한 예치자들이 얻는 이자 수익은 바로 이 대출자들이 지불하는 이자를 재원으로 한다.

- Collateral

  컴파운드 프로토콜에서 가상자산을 대출하려면, 우선 다른 종류의 가상자산을 담보로 제공해야 한다.
  이렇게 담보로 제공된 자산에 대해서도 예치된 자산과 마찬가지로 이자가 축적되며???, cToken이 발행된다.(mint)
  다만 이 자산이 '담보'로써 존재하는 동안은, 즉 대출이 상환되기 전까지는 cToken을 반환하여 원리금을 얻을 수 없다.

- Collateral Factor

  사용자가 대출할 수 있는 최대 금액은 사용자가 제공한 담보에 달려 있다.
  예를 들어 Collateral Factor(담보율)가 70%라면, 내가 제공한 담보의 70%값어치만큼 대출할 수 있는 것
  이 Collateral Factor는 'Comptroller'컨트랙트로 조회가 가능하다

- Enter Markets

  자산을 예치하는 것이 곧 담보의 제공을 의미하는 것은 아니다.
  예치한 자산을 담보로 전환하고자 한다면 시장에 진입?(enter market)해야 한다.

- Calculating Account Liquidity

  Comptroller 컨트랙트는 사용자 계정의 유동성을 계산해주는 함수를 제공한다.
  사용자 계정의 유동성이라 함은 대출 가능한 최대 금액(USD기준)이다.
  이는 서로 다른 Collateral Factor를 가진 여러 담보물이 있을 때 최대 대출 금액을 계산하기 위함이다.

- The Open Price Feed

  컴파운드는 지원되늠 모든 가상 자산에 대한 현재의 exchange rate을 가지고 있는 Price Feed 컨트랙트를 소유하고 있다.
  이 Price Feed는 Chainlink에 의해 동작한다.
  이를 통해 최대 대출액이나 계정의 파산여부를 계산할 수 있는 것이다.

- The Borrow Balance

  상환해야할 원리금
  각각의 cToken 컨트랙트에서 이를 계산하는 함수를 제공한다.

- The Borrow Rate

  대출한 자산에 대한 이자는 이더리움 블록이 생성될 때마다 해당 계정의 대출 잔액에 대출 이율을 곱한 금액을 누적하여 계산된다.
  그러니 상환하지 않는 이상 상환해야 할 원리금은 점점 늘어난다.

- Liquidation

  대출 잔액(원리금)이 Collateral Factor에 허용된 양을 초과하면 대출자는 파산한다.
  제3자는 이 미지급채무(전부 또는 일부)를 대신 상환하는 조건으로 원 대출자의 담보(전부 또는 일부)를 얻을 수 있다.
  여기에 청산 incentive가 제공되는데, 이는 곧 청산 인센티브만큼 할인된 가격으로 담보물을 매입할 수 있다는 의미이다.

- Repaying a Borrow

  대출자는 상응하는 cToken 컨트랙트의 함수를 호출하여 대출을 상환할 수 있다.
  대출이 상환되면, 담보의 이동 또한 자유로워진다.
  cToken 컨트랙트에는 다른 계정을 대신하여 차입금을 상환하는 함수도 있다.
  (이건 청산에 쓰이는 것인가)

- 대출 절차
  - 담보 제공
    - 한 종류 이상의 가상자산을 담보로 제공(사실상 여기까지는 예치)하여 영수증으로 cToken수령(mint)
      - 담보에도 이자가 쌓인다는 사실
    - Comptroller의 enterMarket함수를 호출하여 제공된 자산을 담보로 전환
  - 대출 금액 산정
    1. Open Price Feed 컨트랙트를 통해 대출할 자산의 USD기준 가격 확인
    2. Comptroller의 getAccountLiquidity를 호출하여 사용자 계정의 유동성, 즉 사용자의 최대 대출 가능액을 확인
    3. 2에서 확인한 금액을 1에서 확인한 금액으로 나누어 최대 대출 가능한 수량 확인
       - UDSC나 USDT는 이미 그 가격이 USD기준이기에 이 작업이 필요없다.
  - 대출
    - 대출 수량/금액을 지정하여 상응하는 cToken 컨트랙트의 borrow 함수 호출
    - 계정이 파산되지 않는 선에서 대출한 자산 활용
    - cToken의 repayBorrow함수를 호출하여 대출 상환
