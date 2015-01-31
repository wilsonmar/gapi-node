# gapi-node
# Objectives of this repo
There are 3 objectives to this repo: 
1. show example and describe a way to call Google API the easiest way from within a node.js website.
2. explore current options to make those calls (the various libraries available), 
3. explain JWT construction internals (base64, signing, etc.) used by the code used in #1 above.
4. show additional variations on use of Google API, such as Reports API to collect and display data Google collects.

# Tasks To Do
- [ ] Objective 2 - Explore libraries (done first to not waste time on objective 1)

- [ ] Objective 1 - Call Google API
  - [x] Create github repo (Wilson)
  - [ ] Create github.io/gapi-node (wilson)
  - [ ] Create local node.js skeleton (Wilson with Abdul)
  - [ ] Add library to call Google API 
  - [ ] Add calls to 

- [ ] Objective 3 - Explain JWT
- [ ] Objective 4 - Reports API calls

# JWS
(https://github.com/brianloveswords/node-jws)
implements the creation of a
JSON Web Signature (JWS) as defined in http://self-issued.info/docs/draft-ietf-jose-json-web-signature.html
JSON Web Signature (JWS) represents content secured with digital signatures or Message Authentication Codes (MACs) using JavaScript Object Notation (JSON) based data structures. Cryptographic algorithms and identifiers for use with this specification are described in the separate JSON Web Algorithms (JWA) specification and an IANA registry defined by that specification. Related encryption capabilities are described in the separate JSON Web Encryption (JWE) specification.

# JWT
JWS is an example of JWT defined in the IETF draft at https://tools.ietf.org/html/draft-ietf-oauth-json-web-token-32.
JSON Web Token (JWT) is a compact, URL-safe means of representing claims to be transferred between two parties.  The claims in a JWT are encoded as a JavaScript Object Notation (JSON) object that is used as the payload of a JSON Web Signature (JWS) structure or as the plaintext of a JSON Web Encryption (JWE) structure, enabling the claims to be digitally signed or MACed and/or encrypted.

