# Core Devs Community Call 49


### Meeting Date/Time: October 29th, 2025, 6:00-7:00 UTC
### Meeting Duration: 60 Mins
### [GitHub Agenda Page](https://github.com/tronprotocol/pm/issues/166)
### Agenda

  - [Syncing the development progress of v4.8.1](https://github.com/tronprotocol/java-tron/issues/6342)
  - [TIP-6780: SELFDESTRUCT only in same transaction](https://github.com/tronprotocol/tips/issues/765)

### Detail

- **Murphy**

    Welcome, everyone, to the 49th TRON core developers community meeting. We have two topics on the agenda today. First, I'd like to ask Neo to provide an update on the development progress for version 4.8.1.

**Syncing the development progress of v4.8.1**

- **Neo**
    
    Certainly. Last week, we completed the standard functional testing. We also initiated a specialized testing round for the libp2p network, which we expect to conclude this week. 
    
    Additionally, during our FullNode synchronization tests, a database comparison tool helped us identify a potential resource management issue in RocksDB. While this doesn't affect nodes under normal operation, it could lead to delayed memory release and a risk of OOM (Out of Memory) under extreme conditions, such as performing extensive database scans and creating a high volume of objects in a short time. The Pull Request addressing this issue is nearly ready, and once it's merged, we will conduct a round of enhanced testing. 
    
    On the documentation front, the initial version has been released and will be further improved as we move forward. We are now preparing to draft the technical deep-dive articles and will need participation from our developers to contribute details for their respective features.

- **Murphy**

    Understood. So, is the release for v4.8.1 still expected in November?

- **Neo**
    
    Yes, it should still be in November, but based on our current trajectory, it will likely be in the mid-to-late part of the month.

- **Murphy**
    
    And is the limited live testing for the ARM architecture also underway?

- **Neo**

    Yes, that testing has begun. About two weeks ago, we began a staged rollout of the FullNode synchronization on several ARM devices, running the tests at different intervals. We also have a few ARM-based FullNodes running on Mainnet. No anomalies have been found so far.

- **Boson**
    
    I'd like to add that there is currently an intermittent hanging issue with the unit tests in our CI process.

- **Neo**
    
    Yes, we're aware of that. In previous tests, we found the probability of it hanging was quite low, but we will be running it more frequently in the coming days to monitor its behavior.
    
- **Murphy**
    
    Alright, let's move on to our second topic. I'd like to invite Aiden to present his analysis of the impacts of the `SELFDESTRUCT` opcode modification.

**Progress Sync on `SELFDESTRUCT` Opcode Modification**

- **Aiden**
    
    I'll provide an update on this task. We recently completed a full scan of all internal transactions. The dataset was massive, which is why we just wrapped it up this week. Our preliminary analysis shows that within the past year's transaction data, there is a significant pattern of "create-and-destroy," where contract creation and the `SELFDESTRUCT` call occur within the same transaction. As per our previous discussions, this execution pattern will not be affected by the new `SELFDESTRUCT` logic.
    
    Our next step is to filter out this unaffected data and then conduct a deeper analysis of how the remaining transactions utilize the `SELFDESTRUCT` opcode. This will allow us to more accurately assess the real-world impact. Therefore, this analysis is still in progress, and we have not yet reached a final conclusion.

- **Patrick**
    
    Is there an update on the technical overview for this change? Is it ready to be published?

- **Aiden**
    
    The final section of the technical overview is still being prepared. It will detail the specific impacts identified from this transaction data analysis, so it can only be completed after the analysis is finished.

- **Murphy**

    Okay. Also, regarding TIP-7702, which was a major topic in our last meeting, given the lack of new developments, we will not be discussing it today. We will revisit the topic once there are clear updates to share. 
    
    Does anyone have any other topics they would like to discuss? If not, our meeting will conclude here. Thank you all for attending.


### Attendance

* Aiden
* Patrick
* Blade
* Boson
* Federico
* Sunny
* Jeremy
* Gordon
* Leem
* Daniel
* Mia
* Neo
* Tina
* Vivian
* Wayne
* Erica
* Murphy

