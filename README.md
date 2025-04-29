// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

/**
 * @title KYC Verification Smart Contract
 * @dev A simple contract for KYC verification on the blockchain
 */
contract KYCVerification {
    // Owner of the contract
    address public owner;
    
    // Data structure for KYC information
    struct KYCData {
        bool isVerified;
        uint256 verificationTimestamp;
        string verificationHash; // Hash of off-chain KYC documents
    }
    
    // Mapping from user address to their KYC data
    mapping(address => KYCData) public kycRegistry;
    
    // Events
    event KYCVerified(address indexed user, uint256 timestamp, string verificationHash);
    event KYCRevoked(address indexed user, uint256 timestamp);
    
    // Modifier to restrict access to owner only
    modifier onlyOwner() {
        require(msg.sender == owner, "Only the owner can perform this action");
        _;
    }
    
    /**
     * @dev Constructor to set the owner of the contract
     */
    constructor() {
        owner = msg.sender;
    }
    
    /**
     * @dev Function to verify KYC for a user
     * @param _user Address of the user to be verified
     * @param _verificationHash Hash of the user's KYC documents stored off-chain
     */
    function verifyKYC(address _user, string memory _verificationHash) external onlyOwner {
        require(_user != address(0), "Invalid address");
        require(bytes(_verificationHash).length > 0, "Verification hash cannot be empty");
        
        kycRegistry[_user] = KYCData({
            isVerified: true,
            verificationTimestamp: block.timestamp,
            verificationHash: _verificationHash
        });
        
        emit KYCVerified(_user, block.timestamp, _verificationHash);
    }
    
    /**
     * @dev Function to revoke KYC verification for a user
     * @param _user Address of the user whose verification is to be revoked
     */
    function revokeKYC(address _user) external onlyOwner {
        require(_user != address(0), "Invalid address");
        require(kycRegistry[_user].isVerified, "KYC not verified for this user");
        
        kycRegistry[_user].isVerified = false;
        
        emit KYCRevoked(_user, block.timestamp);
    }
    
    /**
     * @dev Function to check if a user's KYC is verified
     * @param _user Address of the user to check
     * @return bool indicating if the user is verified
     */
    function isKYCVerified(address _user) external view returns (bool) {
        return kycRegistry[_user].isVerified;
    }
}
