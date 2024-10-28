# Leadership Principles


## Bias for Action
At AWS, I was the BMC/firmware tech lead for a product that was supposed to launch in 12 months.  
However due to resourcing constraints, the team needed to me to focus my efforts on the current-year product that was already late.
I was asked to effectively double-duty my work, which is generally generally impossible given the scope I was given (effectively needed ~4 engineers).

The scope of the project involved bringing up a brand new SoC and bootstrapping Linux and UBoot on it, while adding and porting customer-facing features.

Despite these challeges, I was able to make significant progress by:
* Being able to increase the scope of my work on the current-year product.
  I was able to scope and backport requirements from future-product into current-product so that engineering effort didn't need to be repeated.
* I was able to leverage our hardware-design partners with specific testing asks so that I was able test the hardware, even lacking firmware support.
  By doing so I was able to de-risk hardware re-spins, which was appreciated with my stakeholders and customers.
* I was able to structure the project in a way that I owned and completed all tasks that were serial in nature (e.g. board bringup), while deferring tasks that were resource intensive in nature.
  Once current-year project had launched, I was able to earn management trust to be given more engineers whom I was able to parrellelize.
  I also earned trust of my fellow engineers by gathering requirements and doing the scoping for them.
  In short, they already had a bedrock of specifications to execute against when they worked under my direction.

The results of my effort were the following:
* The project was delivered largely almost in-time (mostly delayed due to late feature-adds).
* The risks I took paid off as we didn't have to respin the hardware, keeping the larger program on track.
* Engineers that worked with me were given a sandbox to succeed in, and one of them were promoted after the project.


## Dive Deep
At AWS, I was the tech lead of a controller called a Baseboard Management Controller (BMC), which is an always-on controller present in most Servers/computing devices. 
I owned a new SoC that I had brought up and was the first in productizing it.
One of the unique aspects of a BMC is that it needs almost 100% uptime, as it's primary function is powering on a server.
Unfortunately as part of program, we discovered that in many cases the new SoC wouldn't power on (~5% repro rate), which has massive cascading affects of making the server entirely unavailable

The impact of this was generally two fold:
1. In production, it made the server unsellable to customers (i.e. effective bricks in a datacenter)
2. In manufacturing, it caused turmoil as it required humans to manually power cycle the server.
   Besides the cost of manual labor, this reduced build yields.

This was difficult to solve because there were no interfaces to debug the BMC, aside for network interfaces which were down in these instances.
I made progress by doing the following:
* I enlisted our hardware partners in starting testing at scale on servers with debug probes (UART headers etc.)
* I simulated failures of various hardware components in Qemu to see how our firmware behaved

Through my experiments, I found a few things:
1. We did a lot of hardware initialisation prior to bring up our network interfaces
2. We weren't starting our watchdog early enough

As a fix I patched up our bootloader to kick the watchdog.
After I deferred the bulk of hardware initialisation after network setup, I was able to change the nature of the bug where my SoC was not fully functional but had network connectivity.
This let me debug more effectively and find a poorly written software that went into a infinite loop when hardware didn't behave perfectly.

While I was able to fix this bug, the bug had impacted our customer and the effort of iteratively debugging had slowed down the product schedule.

To address this (and empowered as a lead), I drove enhancements:
* I implemented a serial driver in Linux that tunneled logs into another always-on-controller on the board via a side-band interface.
  This improved the operational posture of our firmware in production where debug probes are not available.
* I drove changes in future hardware that allowed better robustness of debugging than a software driver.

The above has already paid dividends as my team has been able to identify rootcauses quicker, not block their programs.

## Invent and Simplify
Servers often include an EEPROM that essentially holds a manifest for each server (such as serial number, server model, etc.).
This EEPROM is typically authored by various hardware partners involved in the server's manufacturing. 

While supporting operations for my team, I observed that the organization was negatively impacted by the poor quality of data within this EEPROM. 
Issues included parsing errors caused by the lack of a formal specification, which led to server ingestion challenges for Data Center Operations (DCO). 
It also forced teams that rely on this data to take a reactive approach to addressing errors.

To address these issues, I proactively took the lead in developing detailed specifications for the EEPROM content.
Additionally, I created qualification tests to be run in the factory to enforce these specifications and prevent data quality issues from arising.

This initiative also allowed me to proactively address concerns from Supply Chain teams regarding the diversity of components in the Bill of Materials (BOM). 
I included mandates in the specifications for hardware partners to document relevant aspects of the BOM within the EEPROM.

This improvement proved highly valuable during operations, as it enabled teams to identify populated components on a server during component failures without needing to guess. 
It also allowed us to derive metrics on component diversity, which helped reduce supplier risk. 
Ultimately, this effort completely eliminated server ingestion errors for DCO.
