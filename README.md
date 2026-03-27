// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract MyToken {

    // === PODSTAWOWE DANE ===
    string public name = "MyToken";
    string public symbol = "MTK";
    uint8 public decimals = 18;
    uint256 public totalSupply;

    // === WŁAŚCICIEL ===
    address public owner;

    // === MAPOWANIA ===
    mapping(address => uint256) public balanceOf;
    mapping(address => mapping(address => uint256)) public allowance;

    // === PAUZA ===
    bool public paused = false;

    // === EVENTY ===
    event Transfer(address indexed from, address indexed to, uint256 value);
    event Approval(address indexed owner, address indexed spender, uint256 value);
    event Paused();
    event Unpaused();

    // === MODIFIERY ===
    modifier onlyOwner() {
        require(msg.sender == owner, "Nie jestes wlascicielem");
        _;
    }

    modifier whenNotPaused() {
        require(!paused, "Kontrakt jest wstrzymany");
        _;
    }

    // === KONSTRUKTOR ===
    constructor(uint256 _initialSupply) {
        owner = msg.sender;
        totalSupply = _initialSupply * 10 ** decimals;
        balanceOf[msg.sender] = totalSupply;
        emit Transfer(address(0), msg.sender, totalSupply);
    }

    // === TRANSFER ===
    function transfer(address _to, uint256 _value) public whenNotPaused returns (bool) {
        require(balanceOf[msg.sender] >= _value, "Brak srodkow");

        balanceOf[msg.sender] -= _value;
        balanceOf[_to] += _value;

        emit Transfer(msg.sender, _to, _value);
        return true;
    }

    // === APPROVE ===
    function approve(address _spender, uint256 _value) public whenNotPaused returns (bool) {
        allowance[msg.sender][_spender] = _value;

        emit Approval(msg.sender, _spender, _value);
        return true;
    }

    // === TRANSFER FROM ===
    function transferFrom(address _from, address _to, uint256 _value) public whenNotPaused returns (bool) {
        require(balanceOf[_from] >= _value, "Brak srodkow");
        require(allowance[_from][msg.sender] >= _value, "Brak allowance");

        balanceOf[_from] -= _value;
        balanceOf[_to] += _value;
        allowance[_from][msg.sender] -= _value;

        emit Transfer(_from, _to, _value);
        return true;
    }

    // === MINT (tworzenie nowych tokenów) ===
    function mint(address _to, uint256 _amount) public onlyOwner {
        uint256 amount = _amount * 10 ** decimals;

        totalSupply += amount;
        balanceOf[_to] += amount;

        emit Transfer(address(0), _to, amount);
    }

    // === BURN (spalanie tokenów) ===
    function burn(uint256 _amount) public {
        uint256 amount = _amount * 10 ** decimals;

        require(balanceOf[msg.sender] >= amount, "Za malo tokenow");

        balanceOf[msg.sender] -= amount;
        totalSupply -= amount;

        emit Transfer(msg.sender, address(0), amount);
    }

    // === PAUZA ===
    function pause() public onlyOwner {
        paused = true;
        emit Paused();
    }

    function unpause() public onlyOwner {
        paused = false;
        emit Unpaused();
    }

    // === ZMIANA WŁAŚCICIELA ===
    function transferOwnership(address newOwner) public onlyOwner {
        require(newOwner != address(0), "Zly adres");
        owner = newOwner;
    }
}
