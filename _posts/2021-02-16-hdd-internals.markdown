---
layout: default
date:   2021-02-16
title: An introduction to disk drive modeling
tags: storage-systems hard-disk-drives HDD 
---

<h1> An introduction to disk drive modeling </h1>

[‘An introduction to disk drive modeling’](https://pages.cs.wisc.edu/~remzi/Classes/838/Fall2001/Papers/diskmodel-computer94.pdf){:target="\_blank"}
 by Ruemmler and Wilkes published in IEEE
Computer 1994 describes the components and workings of hard disk drives (HDDs)
with the aim of improving disk simulations. Over the years, the workings of
HDDs have stayed qualitatively similar even in the face of orders of magnitude
improvements in the quantitative aspects (e.g., capacity, throughput). This
post discusses some of the interesting and pertinent aspects of HDD internals
as described in the paper, while also borrowing briefly from other related
works [1, 2].  


HDDs consist of multiple components: a data recording and
accessing mechanism (spinning disks and heads), a positioning mechanism that
moves the head to access the data, a mechanism to ensure that the head stays in
the correct position, buffer memory, a controller that acts as a go-between the
host and the disk's data access mechanisms, and a bus interface to the host.



<figure class="caption">
  <img src="{{site.url}}/images/hdd-internal.png"/>
  <figcaption>Figure from the paper showing recording and accessing 
	mechanisms (platters, heads, arm assembly) of a HDD.</figcaption>
</figure>



The recording component of the disk consists of multiple spinning disks called
platters. Each platter is coated with a magnetic material. The orientation of
the magnetic field on the platter encodes the data (0 or 1). Each platter has a
corresponding head for reading and writing data from the platter. To read the
data on a track, the head senses the analog magnetic field on the platter,
converts it to a digital bit stream based on the direction of the magnetic
field, and sends it to the controller. Writing data on the track follows the
reverse path, with the head manipulating the orientation of the magnetic field
on the platter.  

The platters continuously spin around a spindle at a fixed
speed (e.g., 7200 rpm) and consist of multiple concentric tracks. Each track is
further divided into sectors. All sectors in a disk are of the same data size
(e.g. 512 bytes) and the controller reads or writes data at the granularity of
a sector. To access a sector, the corresponding platter's head is first moved
to the corresponding track. This positioning of the head, called seeking, is
performed around a pivot that connects to the head via an arm.  

A seek consists
of four phases: a speed up to a certain speed or distance, coasting at a
constant speed, slow down as the head gets close to the track, and settling on
the exact track (this is required because of high track density, discussed
below). The total time it takes to seek to a track and its breakdown across the
four components is a function of the seek distance. Once the head is positioned
over a track, it waits for and subsequently accesses data from the required
sector as the platter spins underneath it. Thus the latency to access data
depends on the seek time and the rotational speed of the platter. Once the head
is located above the required sector, the data access rate (throughput) is
determined by the rotational speed of the platter.


<figure class="caption">
  <img src="{{site.url}}/images/hdd-servo-bursts.png"/>
  <figcaption>Track-identifying information, called servo bursts, is interspersed throughout the track.</figcaption>
</figure>


Seeking to and staying on the required track is challenging because of high
track density and the minute but non-negligible imperfections in the track
layout (e.g., non-circular or non-concentric tracks). To help with this,
track-identifying information, called servo bursts, is interspersed throughout
the track. Servo bursts are predetermined per-track bit sequences, something
like "I am track 7!". The head continuously reads these servo bursts and sends
them to the controller, which moves the head as required. To actuate a head
movement, the controller applies power to the pivot motor. The amount and
duration of power to apply is encoded in the controller as a function of the
seek distance.  

HDDs expose their addressable space as consecutive blocks and
the controller maps these logical blocks to sectors. The controller lays out
consecutive data blocks over consecutive sectors in a track, over the same
track in consecutive platters, which is called a cylinder, and finally, over
consecutive cylinders. At any given point, only a single head accesses data
from a single platter. Although accessing data simultaneously from a cylinder
using all the heads is possible, it is rarely used because of the challenges in
positioning all the heads at the same exact track in each platter. Servo bursts
are used to switch heads in a cylinder and the settling portion of a seek is
used to move the head to an adjacent track.  

Accessing data sequentially from a
HDD is faster than accessing it randomly; random accesses require seeking to
random tracks whereas sequential accesses are served from the same track or
tracks in the same or adjacent cylinders and require minimal seeking.  

An
optimization to further improve sequential access performance is track skewing.
The starting sector in adjacent tracks are offset by a skew factor. The skew
factor is set such that in the expected time for a track switch, the platters
spin just enough to bring the head to the starting sector of the new track.
With such a skew, the head can continue accessing data sequentially after
moving to an outer track. If the starting sectors of adjacent tracks were
aligned, the head would have to wait for almost an entire rotation after moving
to an outer track to continue accessing data.  

An interesting effect of the
platter's shape is that accessing data along the outer tracks is faster than
accessing data from inner tracks. To understand the reasoning behind this,
first consider that the data density (i.e., data per unit length) is kept
near-constant to its highest possible value to maximize the HDD capacity.
Further, consider that outer tracks are longer than inner tracks. Thus outer
tracks pack more data than inner tracks. As a result, for a given amount data,
the head has to perform fewer seeks (which leads to faster accesses) when
accessing data from outer tracks.

<figure class="caption">
  <img src="{{site.url}}/images/hdd-zones.png"/>
  <figcaption>Zoning: outer zones have more sectors per track and provide faster access.</figcaption>
</figure>

The longer outer tracks also have more sectors per track because sectors are of
fixed data sizes. The hard drive divides tracks into zones, with tracks in the
same zone having the same number of sectors per track. Why zones and why not
variable number of sectors per track? This is because extra track length
between any two consecutive tracks may not be enough to pack an entire sector's
worth of data. This is also the reason why the data-density is near-constant
and not constant.


HDDs use the on-disk buffer memory to cache writes and read-ahead to mask the
seeking latency and rotation-speed-limited data access rate. Write-back caching
enables absorbing a burst of writes without becoming bottle-necked on the
platters. With write-back caching, the disk can acknowledge a write as
completed after committing it only to the cache (buffer memory) and write it to
the backing location (disk platter) at a later point. This poses interesting
trade-offs regarding data durability, e.g., what if the data is lost upon a
power failure (buffer memory is volatile). Read-ahead refers to reading more
than the requested data (e.g., reading an entire track when only a sector was
requested) and caching it so that subsequent sequential reads can be served
from the cache. Read-ahead also presents interesting trade-offs regarding the
value of the read-ahead size in comparison to other uses of the limited buffer
memory size (e.g., write-back caching)

The paper makes for a great read and discusses many other aspects like the bus
interface between the host and HDD, sparing, typical seek lengths and their
power requirements, and the metrics to consider when building a HDD simulator.
I'll end this summary with an observation, rather interesting in hindsight,
that the paper refers to 'hard disk drives' as just 'disk drives', because only
few were thinking or talking about solid-state disk drives (SSDs) in 1994!


References
<ol>
<li> <a href="https://www.usenix.org/conference/fast-03/more-interface—scsi-vs-ata" target="_blank">More than just an interface — SCSI vs ATA</a>, 
Anderson, Dykes, and Riedel, FAST 2003 </li>
<li> <a href="https://pages.cs.wisc.edu/~remzi/OSTEP/file-disks.pdf" target="_blank">Hard Disk Drives</a>, 
Arpaci-Dusseau and Arpaci-Dusseau, <a href="https://pages.cs.wisc.edu/~remzi/OSTEP/" target="_blank">Operating Systems: Theee Easy Pieces (OSTEP) v1</a>, 2018</li>
</ol>
