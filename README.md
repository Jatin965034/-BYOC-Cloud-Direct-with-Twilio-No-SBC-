# -BYOC-Cloud-Direct-with-Twilio-No-SBC-
This project demonstrates BYOC Cloud Direct (Peer-to-Peer) integration between Genesys Cloud and Twilio carrier without using SBC
## Components
Twilio Elastic SIP Trunk with DID(s)
Genesys Cloud BYOC / External Trunk via Genesys Edge
Public Internet (no SBC)
Genesys IVR / Architect flows and Agent routing

## On Twilio — create an Elastic SIP Trunk and bind the DID

In Twilio Console → Elastic SIP Trunking → Create new Trunk. Name it (e.g., genesys-p2p-trunk). Twilio
Under Numbers attach the DID (TWILIO_DID) to the trunk so incoming PSTN calls to that DID are sent into the trunk. Twilio
Configure Origination (where Twilio sends inbound INVITEs):
Add an Origination URI pointing to your Genesys Edge public IP/FQDN and port (use the termination domain/URI Twilio expects). For TCP/UDP use port 5060. If Twilio asks for a next-hop domain/URI, provide the exact FQDN or termination URI Twilio shows for your trunk.
Save the trunk. Note the Termination / Origination URIs Twilio shows — you’ll use these in Genesys.

## In Genesys Cloud — create a BYOC / External Trunk that points to Twilio
Admin → Telephony → Trunks (or BYOC / External Trunks). Select Create Trunk and choose BYOC Cloud (or External Trunk) type. Genesys Cloud Resource Center
Inbound SIP Termination Identifier:
Enter a unique identifier value (this becomes the termination identifier Genesys expects in inbound INVITEs so it can route the call to your org). Genesys requires this to disambiguate traffic from carriers. Make a note of the exact value you enter.
Inbound SIP Termination Header:
Specify which header Twilio (or the carrier) will include that contains your identifier (for BYOC Carrier you typically set a header like X-MyTermination — Genesys docs show examples; Twilio can be configured to pass custom headers in certain cases, or you can rely on the Request-URI if Twilio sends the DID in the Request-URI). Genesys provides options: Request-URI match, header match, or P-Asserted-Identity. Use whichever matches how Twilio sends INVITEs.
Termination & Originator:
In Genesys trunk settings, map Termination Identifier (above) for inbound calls. Set the Outbound/Termination configuration so Genesys will send outbound INVITEs to your Twilio Trunk Termination URI or the provided Twilio domain (use the Twilio termination URI / domain you saw in the Twilio console). Use TCP or UDP (port 5060) as transport — choose TCP if you prefer connection orientation; UDP may be allowed but has higher packet-loss risk on poor networks.
Save the trunk. Genesys will create DNS entries or expect INVITEs with your identifier to reach your org. (If unsure which header/method Twilio uses, consult Twilio Origination settings and Twilio Debug logs; Twilio commonly places the called number in Request-URI/To headers.)

## Inbound tests
From a PSTN phone, call the Twilio DID.
In Twilio Console → Debug / SIP Logs, verify Twilio forwarded an INVITE to your Edge (showing the Request-URI and headers). Twilio
In Genesys Edge logs, verify the INVITE arrived, the termination identifier was detected and the call was routed to the IVR.
Answer in IVR/Queue and verify two-way audio.

## Outbound tests
From an agent, place an outbound call to a PSTN number (or to TWILIO_DID itself).
Verify Genesys sends INVITE to your Edge, Edge sends to Twilio termination URI, and Twilio successfully routes to PSTN.
Check Twilio Debug logs for the inbound INVITE and any 4xx/5xx responses.
