# DEX-test-project-Monad-Blockchain

    // SPDX-License-Identifier: MIT
    pragma solidity ^0.8.0;

    import "@openzeppelin/contracts/token/ERC20/IERC20.sol";

    contract SimpleDEX {
    IERC20 public token1;
    IERC20 public token2;
    uint256 public reserve1;
    uint256 public reserve2;

    event LiquidityAdded(address indexed provider, uint256 amount1, uint256 amount2);
    event LiquidityRemoved(address indexed provider, uint256 amount1, uint256 amount2);
    event Swap(address indexed swapper, uint256 amountIn, uint256 amountOut, address tokenIn, address tokenOut);

    constructor(address _token1, address _token2) {
        token1 = IERC20(_token1);
        token2 = IERC20(_token2);
    }

    function addLiquidity(uint256 amount1, uint256 amount2) external {
        require(amount1 > 0 && amount2 > 0, "Invalid amounts");

        token1.transferFrom(msg.sender, address(this), amount1);
        token2.transferFrom(msg.sender, address(this), amount2);

        reserve1 += amount1;
        reserve2 += amount2;

        emit LiquidityAdded(msg.sender, amount1, amount2);
    }

    function removeLiquidity(uint256 amount1, uint256 amount2) external {
        require(amount1 <= reserve1 && amount2 <= reserve2, "Insufficient reserves");

        token1.transfer(msg.sender, amount1);
        token2.transfer(msg.sender, amount2);

        reserve1 -= amount1;
        reserve2 -= amount2;

        emit LiquidityRemoved(msg.sender, amount1, amount2);
    }

    function swap(address tokenIn, uint256 amountIn) external {
        require(amountIn > 0, "Invalid amount");

        IERC20 inputToken = IERC20(tokenIn);
        IERC20 outputToken = (tokenIn == address(token1)) ? token2 : token1;
        uint256 inputReserve = (tokenIn == address(token1)) ? reserve1 : reserve2;
        uint256 outputReserve = (tokenIn == address(token1)) ? reserve2 : reserve1;

        uint256 amountOut = (amountIn * outputReserve) / (inputReserve + amountIn);

        inputToken.transferFrom(msg.sender, address(this), amountIn);
        outputToken.transfer(msg.sender, amountOut);

        if (tokenIn == address(token1)) {
            reserve1 += amountIn;
            reserve2 -= amountOut;
        } else {
            reserve1 -= amountOut;
            reserve2 += amountIn;
        }

        emit Swap(msg.sender, amountIn, amountOut, tokenIn, address(outputToken));
    }
    }
