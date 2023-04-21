// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Insurance {
    address payable public owner;
    uint public policyCount = 0;

    struct Policy {
        uint id;
        string insuredObject;
        uint insuredValue;
        uint premium;
        uint startDate;
        uint endDate;
        address payable beneficiary;
        bool isActive;
    }

    mapping(uint => Policy) public policies;
    mapping(address => uint[]) public userPolicies;

    event PolicyCreated(uint id, string insuredObject, uint insuredValue, uint premium, uint startDate, uint endDate, address beneficiary);
    event PolicyCancelled(uint id);
    event PolicyPayout(uint id, uint payoutAmount);

    modifier onlyOwner() {
        require(msg.sender == owner, "Only the owner can perform this action.");
        _;
    }

    constructor() {
        owner = payable(msg.sender);
    }

    function createPolicy(string memory _insuredObject, uint _insuredValue, uint _premium, uint _duration, address payable _beneficiary) public payable returns (uint) {
        require(msg.value == _premium, "Premium amount must be paid upfront.");

        policyCount++;
        uint endDate = block.timestamp + _duration;

        Policy memory newPolicy = Policy(policyCount, _insuredObject, _insuredValue, _premium, block.timestamp, endDate, _beneficiary, true);
        policies[policyCount] = newPolicy;
        userPolicies[msg.sender].push(policyCount);

        emit PolicyCreated(policyCount, _insuredObject, _insuredValue, _premium, block.timestamp, endDate, _beneficiary);

        return policyCount;
    }

    function cancelPolicy(uint _id) public {
        require(msg.sender == policies[_id].beneficiary, "Only the beneficiary can cancel the policy.");
        require(policies[_id].isActive, "The policy is already cancelled or expired.");

        policies[_id].isActive = false;

        emit PolicyCancelled(_id);
    }

    function claim(uint _id) public {
        require(msg.sender == policies[_id].beneficiary, "Only the beneficiary can claim the policy payout.");
        require(policies[_id].isActive, "The policy is already cancelled or expired.");

        uint payoutAmount = policies[_id].insuredValue;
        policies[_id].isActive = false;
        policies[_id].beneficiary.transfer(payoutAmount);

        emit PolicyPayout(_id, payoutAmount);
    }

    function getPolicyIdsByUser(address _user) public view returns (uint[] memory) {
        return userPolicies[_user];
    }
}
