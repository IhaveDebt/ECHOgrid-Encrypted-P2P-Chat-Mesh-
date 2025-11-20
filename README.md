//
//  ECHOgrid.swift
//

import Foundation
import MultipeerConnectivity
import CryptoKit

final class EchoNode: NSObject, MCSessionDelegate, MCNearbyServiceAdvertiserDelegate {
    
    let peerID = MCPeerID(displayName: UUID().uuidString)
    var session: MCSession!
    var advertiser: MCNearbyServiceAdvertiser!
    
    override init() {
        super.init()
        
        session = MCSession(peer: peerID, securityIdentity: nil, encryptionPreference: .required)
        session.delegate = self
        
        advertiser = MCNearbyServiceAdvertiser(peer: peerID, discoveryInfo: nil, serviceType: "echogrid")
        advertiser.delegate = self
        advertiser.startAdvertisingPeer()
    }
    
    func sendMessage(_ text: String) {
        let key = SymmetricKey(size: .bits256)
        let raw = text.data(using: .utf8)!
        let encrypted = try! AES.GCM.seal(raw, using: key).combined!
        
        try? session.send(encrypted, toPeers: session.connectedPeers, with: .reliable)
    }
    
    func advertiser(_ advertiser: MCNearbyServiceAdvertiser, didReceiveInvitationFromPeer peerID: MCPeerID,
                    withContext context: Data?, invitationHandler: @escaping (Bool, MCSession?) -> Void) {
        invitationHandler(true, session)
    }
    
    func session(_ session: MCSession, didReceive data: Data, fromPeer peerID: MCPeerID) {
        print("Encrypted message received — size: \(data.count) bytes")
    }
    
    // Unused stubs
    func session(_ session: MCSession, peer peerID: MCPeerID, didChange state: MCSessionState) {}
    func session(_ session: MCSession, didStartReceivingResourceWithName resourceName: String,
                 fromPeer peerID: MCPeerID, with progress: Progress) {}
    func session(_ session: MCSession, didReceive stream: InputStream, withName streamName: String,
                 fromPeer peerID: MCPeerID) {}
    func session(_ session: MCSession, didFinishReceivingResourceWithName resourceName: String,
                 fromPeer peerID: MCPeerID, at localURL: URL?, withError error: Error?) {}
}

let node = EchoNode()
RunLoop.main.run()
