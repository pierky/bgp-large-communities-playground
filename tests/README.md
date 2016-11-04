# Implemented Features and Interoperability Tests

I used this Playground to run tests on some of the currently available tools which implement Large BGP Communities. The goal is to fill some blanks and enrich contents on the [IETF idr Implementations of draft-ietf-idr-large-community page]( https://trac.tools.ietf.org/wg/idr/trac/wiki/draft-ietf-idr-large-community%20implementations).

The topics I wanted to focus on here are the following:

- the ability to correctly send, receive and display prefixes carrying large BGP communities - [01-ANNOUNCING_RECEIVING_DISPLAYING.md](01-ANNOUNCING_RECEIVING_DISPLAYING.md).

- the ability to filter prefixes on the basis of large BGP communities and to perform actions on them - [02-OPERATIONS_ON_COMMUNITIES.md](02-OPERATIONS_ON_COMMUNITIES.md).

- the behaviour regarding duplicate large BGP communities handling, both on the sending and the receiving side - [03-DUPLICATE_COMMUNITIES.md](03-DUPLICATE_COMMUNITIES.md).

- error handling of malformed large BGP communities attributes - [04-MALFORMED-ATTRIBUTES.md](04-MALFORMED-ATTRIBUTES.md).

OSS implementations that I tested are those covered by the Playground itself at the moment: ExaBGP, GoBGP and BIRD for the "BGP speakers" category and pmacct for the "Tools / Ecosystem" one. For each topic, configuration files can be found under the respective `tests/` subdirectory.

# Results

Tests that I ran using this *playground* brought me to file some reports and issues to devs teams which quickly followed up with improvements: I'm sure these enhancements would have all been completed soon or later even if this toy had never existed, but I'm satisfied I have speeded them up.

- A *cosmetic* bug in BIRD [has been fixed](https://github.com/BIRD/bird/commit/a46e01eeef17a7efe876618623397f60e62afe37).
- GoBGP added support for large BGP communities on [policies](https://github.com/osrg/gobgp/issues/1133) and improved [the way it handles duplicate values](https://github.com/osrg/gobgp/issues/1143).
- ExaBGP improved the way it handles [duplicate communities](https://github.com/pierky/large-bgp-communities-playground/issues/2) and [malformed attributes](https://github.com/Exa-Networks/exabgp/issues/514) in UPDATEs.
- tcpdump (which is not strictly related to this playground but that I used here anyhow) [added large BGP communities support](https://github.com/the-tcpdump-group/tcpdump/issues/543) to its output.

## Implemented features and compliance

With regards of [Implemented Features of draft-ietf-idr-large-community wiki page]( https://trac.tools.ietf.org/wg/idr/trac/wiki/draft-ietf-idr-large-community%20implementations) the outcome of these tests can be summarized in this way:

* **Send correctly formatted Large Communities**: :white_check_mark: ExaBGP; :white_check_mark: GoBGP; :white_check_mark: BIRD; :x: Quagga.

  Each implementation (except for the [Quagga's patch](https://bugzilla.quagga.net/show_bug.cgi?id=875#c3) I tested - bug 875, comment n. 3) can send prefixes that have been tagged with Large BGP Communities and which can be received and processed by the other implementations. Tests have been done by adding 1, 2 and 3 communities. Quagga could only send one community at once.

* **Receive and display Large Communities with routes**: :white_check_mark: ExaBGP; :white_check_mark: GoBGP; :white_check_mark: BIRD; :white_check_mark: pmacct; :white_check_mark: Quagga.

  Implementations correctly show the large communities next to the prefixes which they are attached to.

* **Match Large Communities using the 3 decimal uint32 representation**: :white_check_mark: ExaBGP; :white_check_mark: GoBGP; :white_check_mark: BIRD; :white_check_mark: Quagga.
* **Set/delete Large Communities using the 3 decimal uint32 representation**: :white_check_mark: ExaBGP; :white_check_mark: GoBGP; :white_check_mark: BIRD; :x: Quagga.

  Some prefixes carrying large BGP communities have been announced to the implementations. GoBGP and BIRD have been able to perform actions on them (to add one or more communities, to delete one or more communities, to match prefixes on the basis of the communities they carried and then to perform actions on them).
  Here ExaBGP is given with a positive outcome because, by its own nature, it allows to implement filtering and manipulation logic within the third party applications it interacts with. Quagga has correctly matched prefixes on the basis of carried large BGP communities and it performed the desired action, but I have not been able to add/delete communities: I have added a remark to the [Quagga's Bug 875](https://bugzilla.quagga.net/show_bug.cgi?id=875) that is used to track its patch.

* **Separator used in the textual representation**: ExaBGP: `:` in log files, `,` in JSON dumps (array); GoBGP: `:`; BIRD: `,`; pmacct: `:`; Quagga: `:`.

* **No restriction on the value of Global Administrator field**: :white_check_mark: ExaBGP; :white_check_mark: GoBGP; :white_check_mark: BIRD; :white_check_mark: pmacct; :white_check_mark: Quagga.

  Tests have been done using the [draft-ietf-idr-large-community-02](https://tools.ietf.org/html/draft-ietf-idr-large-community-02) [Reserved Large BGP Community values](https://tools.ietf.org/html/draft-ietf-idr-large-community-02#section-5) `65535:1:1` and `4294967295:4294967295:4294967295`. All the implementations correctly sent, received and displayed the prefixes which carried these values and also correctly displayed the large communities' values.

* **Textual representation, integers not omitted, even when zero**: :white_check_mark: ExaBGP; :white_check_mark: GoBGP; :white_check_mark: BIRD; :white_check_mark: pmacct; :white_check_mark: Quagga.

  [Section 4](https://tools.ietf.org/html/draft-ietf-idr-large-community-02#section-4) of the draft states that, in textual representation of large communities, `These integers MUST NOT be omitted, even when zero.`. Two communities have been used to test compliance with this statement, `ASN:0:1` and `ASN:1:0`, and both have been displayed correctly.

* **Duplicate Large Communities not transmitted**: :white_check_mark: ExaBGP; :x: GoBGP; :white_check_mark: BIRD; :white_check_mark: pmacct; :white_check_mark: Quagga.

  [Section 2](https://tools.ietf.org/html/draft-ietf-idr-large-community-02#section-2) of draft-ietf-idr-large-community-02 mandates that `Duplicate Large Communities SHOULD NOT be transmitted`. A prefix has been configured with a duplicate community; GoBGP included the duplicate community in its announced prefix.
  BIRD removed the duplicate value from the configured prefix before announcing it; it included the duplicate community only when it propagated the prefix received from GoBGP to ExaBGP. This behavior makes me suppose that it depends on the outcome of the test reported in the next bullet.

* **Removing duplicate Large Communities from received UPDATEs**: :white_check_mark: ExaBGP; :white_check_mark: GoBGP; :x: BIRD; :white_check_mark: pmacct; :white_check_mark: Quagga.

  [Section 2](https://tools.ietf.org/html/draft-ietf-idr-large-community-02#section-2) also states that `A receiving speaker SHOULD silently remove duplicate Large Communities from a BGP UPDATE message.`. Compliance with this statement has been tested by leveraging on the previous bullet, by using GoBGP as sender of duplicate communities. BIRD resulted in not removing duplicate communities on receipt. BIRD's devs have been informed and they agree this behaviour should be improved.

* **Treating as withdraw prefixes contained in UPDATEs that carry malformed attribute**: :white_check_mark: ExaBGP; :x: GoBGP; :white_check_mark: BIRD; :x: pmacct; :white_check_mark: Quagga.

  [Section 6](https://tools.ietf.org/html/draft-ietf-idr-large-community-02#section-6) of draft-ietf-idr-large-community-02 sets the policy that must be followed for error handling of malformed large BGP communities attributes: `A BGP UPDATE message with a malformed Large Communities attribute SHALL be handled using the approach of "treat-as-withdraw" as described in section 2 [RFC7606]`. The [*treat-as-withdraw* approach](https://tools.ietf.org/html/rfc7606#section-2) wants that routes contained in the involved UPDATE message have to be withdrawn "just as if they had been listed in the WITHDRAWN ROUTES field". GoBGP is resulted in closing the BGP session; pmacct reported the prefix included in the malformed UPDATE in its BGP table dump. Issues have been filed to ask support to devs for all the implementations ([GoBGP issue](https://github.com/osrg/gobgp/issues/1147), [pmacct](https://github.com/pmacct/pmacct/issues/65)).

## Overview of interoperability

For what concerns the OSS implementations I tested, the interoperability matrix on the IETF implementations wiki page can be filled with all `yes`.

# Change log

## 2016-11-04

- More tests on Quagga:
 - operations on communities (match/set/delete)
 - duplicate communities handling
 - malformed attributes handling

## 2016-10-31

- First patch for Quagga.

- GoBGP removing duplicate values on receipt.

## 2016-10-26

- Update ExaBGP malformed attributes handling behaviour after [fix](https://github.com/Exa-Networks/exabgp/issues/514).

## 2016-10-20

- Interoperability tests between pmacct and GoBGP/BIRD.

## 2016-10-17

- Add pmacct docker image.

- Tests extended to pmacct.

## 2016-10-14

- New results from tests on malformed large BGP communities attributes handling.

## 2016-10-11

- Add results from a new test about removing duplicate Large Communities from received UPDATEs.

- Move tests about duplicate communities into a separate file.

- Remove test about duplicate communities receivers behaviour (it was [-01, Section 6](https://tools.ietf.org/html/draft-ietf-idr-large-community-01#section-6)).

- Positive outcome for GoBGP, now implementing policies and actions based on large communities (osrg/gobgp#1133).

## 2016-10-10

- Update draft version (-01 to -02).

- Mark as *outdated* a test about duplicate communities receivers behaviour.

- Positive outcome for ExaBGP, no longer sending duplicate communities since [541f0ee01e42d2f191677add2e7914e43c9effc2](https://github.com/Exa-Networks/exabgp/commit/541f0ee01e42d2f191677add2e7914e43c9effc2).

## 2016-10-05

- First release.

- BIRD *cosmetic* bug found and [fixed](https://github.com/BIRD/bird/commit/a46e01eeef17a7efe876618623397f60e62afe37).

- GoBGP enhancement [proposed](https://github.com/osrg/gobgp/issues/1133).

# Future work

* More tests on implementations as they become available: OpenBGPD, Cisco IOS XR, ...

# Credits

Thanks to [@job](https://github.com/job) for inspiration and support, and to [Peter Hessler](https://twitter.com/phessler) for suggesting me the malformed message test.

# Author

Pier Carlo Chiodi - https://pierky.com

Blog: https://blog.pierky.com Twitter: [@pierky](https://twitter.com/pierky)

