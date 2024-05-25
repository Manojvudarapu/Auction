# Auction

// SPDX-License-Identifier: MIT
pragma solidity ^0.8.7;

contract Auction {
    address public owner;
    uint public auctionEndTime;
    uint public highestBid;
    address public highestBidder;
    bool public auctionEnded;

    mapping(address => uint) public bids;

    event NewBid(address indexed bidder, uint amount);
    event AuctionEnded(address winner, uint amount);

    constructor(uint _durationMinutes) {
        owner = msg.sender;
        auctionEndTime = block.timestamp + (_durationMinutes * 1 minutes);
        auctionEnded = false;
    }

    modifier onlyOwner() {
        require(msg.sender == owner, "Only the owner can call this function");
        _;
    }

    modifier onlyBeforeEnd() {
        require(block.timestamp < auctionEndTime, "Auction has already ended");
        _;
    }

    modifier onlyAfterEnd() {
        require(block.timestamp >= auctionEndTime, "Auction has not ended yet");
        _;
    }

    function placeBid() public payable onlyBeforeEnd {
        require(msg.value > highestBid, "Bid must be higher than the current highest bid");

        // Refund the previous highest bidder
        if (highestBidder != address(0)) {
            bids[highestBidder] += highestBid;
        }

        highestBid = msg.value;
        highestBidder = msg.sender;
        bids[msg.sender] += msg.value;

        emit NewBid(msg.sender, msg.value);
    }

    function endAuction() public onlyOwner onlyAfterEnd {
        require(!auctionEnded, "Auction has already been ended");

        auctionEnded = true;
        emit AuctionEnded(highestBidder, highestBid);

        // Transfer the highest bid amount to the owner
        payable(owner).transfer(highestBid);
    }

    function withdrawBid() public onlyBeforeEnd {
        require(msg.sender != highestBidder, "You cannot withdraw the highest bid");
        uint amount = bids[msg.sender];
        require(amount > 0, "No bid to withdraw");
        bids[msg.sender] = 0;
        payable(msg.sender).transfer(amount);
    }
}
