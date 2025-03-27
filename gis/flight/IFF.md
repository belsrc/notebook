---
reference:
  - https://en.wikipedia.org/wiki/Identification_friend_or_foe
  - https://code7700.com/transponder.htm
  - https://en.wikipedia.org/wiki/Aviation_transponder_interrogation_modes
  - https://electronicstechnician.tpub.com/14089/css/Figure-3-2-Aims-Mark-Xii-Iff-Interrogations-And-Replies-45.htm
---
## Modes

- **Mode 1 (Military)** - 2-digit octal (6 bit) "mission code" that identifies the aircraft type or mission. The first digit of the reply code must be a number from 0 to 7. The second digit must be a number from 0 to 3.

- **Mode 2 (Military)** - 4-digit octal (12-bit) unit code or tail number, used to identify individual aircraft. In mode2 reply codes, each of the four reply digits can have any value from 0 to 7.

- **Mode 3 (Military)** / **Mode A (Civilian)** - 4-digit octal (12 bit) code set in the cockpit as assigned by ATC, often called "Mode 3/A" and usually combined with Mode C to provide altitude information. The Department of Defense is assigned four mode 3/A code blocks (50XX, 54XX, 61XX, 64XX) for use within U.S. national air space.

- **Mode 3 (Military)** / **Mode C (Civilian)** - Provides the aircraft's pressure altitude, sometimes called "Mode 3C."

- **Mode 4 (Military)** - Provides a 3-pulse reply to crypto coded challenge. Deprecated.

- **Mode 5 (Military)** - Provides a cryptographically secured version of Mode S and ADS-B GPS position.

- **Mode S (Military and Civilian)** - Provides multiple information formats to a selective interrogation. Each aircraft is assigned a fixed 24-bit address.

### Mode 3/A

- **1200** - Transponder Operation Under Visual Flight Rules (VFR): Unless otherwise instructed by an ATC facility, adjust transponder to reply on Mode 3/A Code 1200 regardless of altitude.

- **7500 Hijacking** - An aircraft equipped with an SSR transponder is expected to operate the transponder on Mode A Code 7500 to indicate specifically that it is the subject of unlawful interference.

- **7600 Lost Comm** - An aircraft equipped with an SSR transponder is expected to operate the transponder on Mode A Code 7600 to indicate that it has experienced air-ground communication failure.

- **7700 Emergency** - The pilot of an aircraft in a state of emergency shall set the transponder to Mode A Code 7700 unless ATC has previously directed the pilot to operate the transponder on a specified code.

- **7777** - Assigned to interceptors on active air defense missions.