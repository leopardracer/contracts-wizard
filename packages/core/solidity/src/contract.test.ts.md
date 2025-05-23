# Snapshot report for `src/contract.test.ts`

The actual snapshot is saved in `contract.test.ts.snap`.

Generated by [AVA](https://avajs.dev).

## contract basics

> Snapshot 1

    `// SPDX-License-Identifier: MIT␊
    // Compatible with OpenZeppelin Contracts ^5.0.0␊
    pragma solidity ^0.8.27;␊
    ␊
    contract Foo {␊
    }␊
    `

## contract name is unicodeSafe

> Snapshot 1

    `// SPDX-License-Identifier: MIT␊
    // Compatible with OpenZeppelin Contracts ^5.0.0␊
    pragma solidity ^0.8.27;␊
    ␊
    contract Footec {␊
    }␊
    `

## contract with a parent

> Snapshot 1

    `// SPDX-License-Identifier: MIT␊
    // Compatible with OpenZeppelin Contracts ^5.0.0␊
    pragma solidity ^0.8.27;␊
    ␊
    import {Bar} from "./Bar.sol";␊
    ␊
    contract Foo is Bar {␊
    }␊
    `

## contract with two parents

> Snapshot 1

    `// SPDX-License-Identifier: MIT␊
    // Compatible with OpenZeppelin Contracts ^5.0.0␊
    pragma solidity ^0.8.27;␊
    ␊
    import {Bar} from "./Bar.sol";␊
    import {Quux} from "./Quux.sol";␊
    ␊
    contract Foo is Bar, Quux {␊
    }␊
    `

## contract with a parent with parameters

> Snapshot 1

    `// SPDX-License-Identifier: MIT␊
    // Compatible with OpenZeppelin Contracts ^5.0.0␊
    pragma solidity ^0.8.27;␊
    ␊
    import {Bar} from "./Bar.sol";␊
    ␊
    contract Foo is Bar {␊
        constructor() Bar("param1", "param2") {}␊
    }␊
    `

## contract with two parents only one with parameters

> Snapshot 1

    `// SPDX-License-Identifier: MIT␊
    // Compatible with OpenZeppelin Contracts ^5.0.0␊
    pragma solidity ^0.8.27;␊
    ␊
    import {Bar} from "./Bar.sol";␊
    import {Quux} from "./Quux.sol";␊
    ␊
    contract Foo is Bar, Quux {␊
        constructor() Bar("param1", "param2") {}␊
    }␊
    `

## contract with one override

> Snapshot 1

    `// SPDX-License-Identifier: MIT␊
    // Compatible with OpenZeppelin Contracts ^5.0.0␊
    pragma solidity ^0.8.27;␊
    ␊
    contract Foo {␊
    }␊
    `

## contract with two overrides

> Snapshot 1

    `// SPDX-License-Identifier: MIT␊
    // Compatible with OpenZeppelin Contracts ^5.0.0␊
    pragma solidity ^0.8.27;␊
    ␊
    contract Foo {␊
        // The following functions are overrides required by Solidity.␊
    ␊
        function _beforeTokenTransfer(address from, address to, uint256 amount)␊
            internal␊
            override(ERC20, ERC20Snapshot)␊
        {␊
            super._beforeTokenTransfer(from, to, amount);␊
        }␊
    }␊
    `

## contract with two different overrides

> Snapshot 1

    `// SPDX-License-Identifier: MIT␊
    // Compatible with OpenZeppelin Contracts ^5.0.0␊
    pragma solidity ^0.8.27;␊
    ␊
    contract Foo {␊
        // The following functions are overrides required by Solidity.␊
    ␊
        function _beforeTokenTransfer(address from, address to, uint256 amount)␊
            internal␊
            override(ERC20, OtherParent)␊
        {␊
            super._beforeTokenTransfer(from, to, amount);␊
        }␊
    ␊
        function _otherFunction() internal override(ERC20, OtherParent) {␊
            super._otherFunction();␊
        }␊
    }␊
    `

## contract with a modifier

> Snapshot 1

    `// SPDX-License-Identifier: MIT␊
    // Compatible with OpenZeppelin Contracts ^5.0.0␊
    pragma solidity ^0.8.27;␊
    ␊
    contract Foo {␊
        function _otherFunction() internal whenNotPaused {}␊
    }␊
    `

## contract with a modifier and override

> Snapshot 1

    `// SPDX-License-Identifier: MIT␊
    // Compatible with OpenZeppelin Contracts ^5.0.0␊
    pragma solidity ^0.8.27;␊
    ␊
    contract Foo {␊
        function _otherFunction() internal override(ERC20, OtherParent) whenNotPaused {␊
            super._otherFunction();␊
        }␊
    }␊
    `

## contract with constructor code

> Snapshot 1

    `// SPDX-License-Identifier: MIT␊
    // Compatible with OpenZeppelin Contracts ^5.0.0␊
    pragma solidity ^0.8.27;␊
    ␊
    contract Foo {␊
        constructor() {␊
            _mint(msg.sender, 10 ether);␊
        }␊
    }␊
    `

## contract with constructor code and a parent

> Snapshot 1

    `// SPDX-License-Identifier: MIT␊
    // Compatible with OpenZeppelin Contracts ^5.0.0␊
    pragma solidity ^0.8.27;␊
    ␊
    import {Bar} from "./Bar.sol";␊
    ␊
    contract Foo is Bar {␊
        constructor() Bar("param1", "param2") {␊
            _mint(msg.sender, 10 ether);␊
        }␊
    }␊
    `

## contract with function code

> Snapshot 1

    `// SPDX-License-Identifier: MIT␊
    // Compatible with OpenZeppelin Contracts ^5.0.0␊
    pragma solidity ^0.8.27;␊
    ␊
    contract Foo {␊
        function _otherFunction() internal {␊
            _mint(msg.sender, 10 ether);␊
        }␊
    }␊
    `

## contract with overridden function with code

> Snapshot 1

    `// SPDX-License-Identifier: MIT␊
    // Compatible with OpenZeppelin Contracts ^5.0.0␊
    pragma solidity ^0.8.27;␊
    ␊
    contract Foo {␊
        function _otherFunction() internal override {␊
            _mint(msg.sender, 10 ether);␊
            super._otherFunction();␊
        }␊
    }␊
    `

## contract with one variable

> Snapshot 1

    `// SPDX-License-Identifier: MIT␊
    // Compatible with OpenZeppelin Contracts ^5.0.0␊
    pragma solidity ^0.8.27;␊
    ␊
    contract Foo {␊
        uint value = 42;␊
    }␊
    `

## contract with two variables

> Snapshot 1

    `// SPDX-License-Identifier: MIT␊
    // Compatible with OpenZeppelin Contracts ^5.0.0␊
    pragma solidity ^0.8.27;␊
    ␊
    contract Foo {␊
        uint value = 42;␊
        string name = "john";␊
    }␊
    `

## name with special characters

> Snapshot 1

    `// SPDX-License-Identifier: MIT␊
    // Compatible with OpenZeppelin Contracts ^5.0.0␊
    pragma solidity ^0.8.27;␊
    ␊
    contract FooBarBaz {␊
    }␊
    `

## contract with info

> Snapshot 1

    `// SPDX-License-Identifier: MIT␊
    // Compatible with OpenZeppelin Contracts ^5.0.0␊
    pragma solidity ^0.8.27;␊
    ␊
    /// @custom:security-contact security@example.com␊
    contract Foo {␊
    }␊
    `
