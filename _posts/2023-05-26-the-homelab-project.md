---
layout: post
title: "The homelab project"
author: "Carlos Camacho"
categories:
  - blog
tags:
  # - draft
  - cloud
  - kubernetes
  - engineering
  - hobbies
# hidden: true
favorite: true
commentIssueId: 90
refimage: '/static/homelab/00_intro/01_homelab.jpg'
---

Homelabs, also known as home data center, refers to a personal setup created by
technology enthusiasts or professionals in their homes for learning, experimentation,
or production purposes. It typically consists of a collection of computer hardware,
networking equipment, and software that simulates or replicates a professional IT environment.

Homelabs serve various purposes, including:

- Learning and Skill Development: Many individuals use homelabs to enhance their
knowledge and skills in areas such as system administration, networking, virtualization,
storage, and cybersecurity. It provides a hands-on environment to experiment with different
technologies, configurations, and software without the constraints of a production environment.

- Testing and Prototyping: Homelabs offer a safe and controlled environment to test new software,
applications, or hardware configurations. It allows individuals to evaluate their performance,
compatibility, and feasibility before deploying them in a production environment.

- Personal Projects and Services: Some people build homelabs to host personal projects, services,
or applications. For example, they may set up media servers, game servers, file sharing platforms,
or websites to meet their specific needs or interests.

- Home Automation and Internet of Things (IoT): Homelabs can be utilized for setting up smart home
automation systems and experimenting with IoT devices. This enables individuals to control and
manage various aspects of their home, such as lighting, temperature, security, and entertainment systems.

- Data Storage and Backup: Homelabs often include storage solutions like Network-Attached Storage
(NAS) or storage area network (SAN) devices, which allow users to store and backup their data locally.

Homelabs can range from a single server or a few network devices to a comprehensive setup with
multiple servers, switches, routers, and other equipment. They can be built using off-the-shelf
components or repurposed enterprise-grade hardware, depending on the user's requirements, budget,
and level of expertise.


![](/static/homelab/00_intro/00_homelab.jpg)

While building and maintaining a homelab can be challenging, it can also be a rewarding and educational experience.
The availability of upstream communities and resources can help you overcome difficulties and find support from
other homelab enthusiasts.
Start with a clear plan, research thoroughly, and take it step by step, adjusting your goals and complexity as you
gain experience.

## Origins

<div style="float: left; width: 230px; background: white;"><img src="/static/homelab/00_intro/02_humble_home_lab.jpg" alt="" style="border:15px solid #FFF"></div>
Datacenters are large-scale facilities designed to house and manage extensive computing infrastructure 
for businesses or organizations. They have high-end server racks, robust networking equipment, redundant
power and cooling systems, and advanced security measures. Datacenters offer scalability, reliability,
and support for critical operations, often involving multiple clients or users. In contrast, homelabs
are personal setups in individuals' homes used for learning, experimentation, or small-scale production.
They typically consist of a collection of hardware, networking devices, and software. Homelabs provide a
controlled environment for skill development, testing, and hosting personal projects, but on a smaller
scale with limited resources and fewer users.

<div style="float: right; width: 230px; background: white;"><img src="/static/homelab/00_intro/03_ideal_rack.jpg" alt="" style="border:15px solid #FFF"></div>
The ideal homelab architecture typically includes a combination of components to create a versatile and
powerful setup. It starts with robust server hardware, whether it's rack-mounted servers or repurposed 
desktop machines, equipped with ample processing power, memory, and storage capacity. Networking equipment
such as routers, switches, and firewalls play a crucial role in connecting devices and enabling communication
within the homelab. Virtualization software like VMware or Hyper-V allows for the creation and management of
virtual machines, enabling efficient resource utilization. Storage solutions, including NAS
(Network Attached Storage) or SAN (Storage Area Network), provide ample storage capacity for data and backups.
Additionally, monitoring and management tools help ensure the stability and performance of the homelab environment.
The ideal architecture emphasizes flexibility, scalability, and the ability to experiment with various technologies
and configurations to meet specific learning or project requirements.

<div style="float: left; width: 230px; background: white;"><img src="/static/homelab/00_intro/04_networking.png" alt="" style="border:15px solid #FFF"></div>
Networking is a crucial component in an ideal homelab architecture, enabling devices to communicate and share resources
effectively. It starts with a reliable and feature-rich router that provides internet access and manages IP addresses.
A network switch connects multiple devices within the homelab, allowing seamless communication. Managed switches offer
advanced features for improved network performance and segmentation. Implementing a firewall ensures the security of the
homelab, protecting against unauthorized access. Additionally, utilizing technologies like VLANs and QoS can enhance
network efficiency and prioritize traffic. Proper networking setup enables connectivity, facilitates the sharing of
services and resources, and creates a robust foundation for various homelab activities and projects.

<div style="float: right; width: 230px; background: white;"><img src="/static/homelab/00_intro/05_storage.jpg" alt="" style="border:15px solid #FFF"></div>

Storage is a critical component in an ideal homelab architecture, providing ample space to store data, virtual machine
images, and backups. Network Attached Storage (NAS) or Storage Area Network (SAN) solutions offer centralized storage with
high capacity and performance. NAS devices are easy to set up and provide shared file access over the network, making them
suitable for storing media, documents, and backups. SANs, on the other hand, deliver fast and reliable storage for virtual machine environments,
supporting features like RAID, snapshotting, and replication. Additionally, leveraging cloud storage services can provide off-site backups and enhance
data accessibility. By incorporating a robust storage solution into the homelab architecture, users can ensure data integrity, scalability, and
efficient management of their digital assets.

<div style="float: left; width: 230px; background: white;"><img src="/static/homelab/00_intro/06_compute.jpg" alt="" style="border:15px solid #FFF"></div>
Compute is a fundamental aspect of an ideal homelab architecture, empowering users to run diverse workloads and applications. It typically revolves
around powerful server hardware, which can include rack-mounted servers or repurposed desktop machines with ample processing power, memory, and storage
capacity. Virtualization technologies like VMware or Hyper-V allow for the creation and management of virtual machines, enabling efficient utilization of
resources and the ability to run multiple operating systems and applications simultaneously. Additionally, containerization platforms such as Docker and
Kubernetes offer lightweight and scalable environments for deploying and managing containerized applications. The compute component of a homelab provides
the necessary horsepower to support a wide range of experiments, projects, and learning opportunities, empowering users to explore various technologies
and configurations.

<div style="float: right; width: 230px; background: white;"><img src="/static/homelab/00_intro/07_nice_homelab.jpg" alt="" style="border:15px solid #FFF"></div>
A cool homelab is a sight to behold, with a sleek equipment rack housing powerful servers and illuminated by LED lights. Neatly managed cables run along
cable management arms and trays, with color-coded sleeves adding style and clarity. High-performance networking switches, routers, and firewalls display
blinking status lights, creating an entrancing visual display. Multiple monitors on a spacious desk provide a command center, surrounded by shelves and
storage units for additional hardware. The room is optimized for airflow and ventilation, ensuring the equipment remains cool and efficient. Whiteboards
or smart boards adorn the walls, inspiring creativity and organization. This cool homelab is a blend of functionality, organization, and aesthetic
appeal, reflecting a passion for technology and a commitment to continuous learning and exploration.

<div style="float: left; width: 230px; background: white;"><img src="/static/homelab/00_intro/08_reality_mess.jpg" alt="" style="border:15px solid #FFF"></div>
Keeping things organized in a homelab can be a real challenge, but it's crucial for maintaining efficiency and ease of use. One area that often requires
attention is cable management. With numerous devices, power cords, and networking cables, it's easy for things to become a tangled mess. Implementing
cable management techniques such as using cable ties, Velcro straps, or cable management trays can help keep cables organized and prevent them from
becoming a jumbled mess. Additionally, labeling cables and ports can make it easier to trace connections and troubleshoot any issues that may arise.
Regular maintenance and periodic cable cleanups can go a long way in maintaining a tidy and well-organized homelab.

<div style="float: right; width: 230px; background: white;"><img src="/static/homelab/00_intro/09_reality_mess.jpg" alt="" style="border:15px solid #FFF"></div>
Aside from cables, proper equipment and component organization also contribute to an organized homelab. Utilizing equipment racks or shelves can provide
a designated space for servers, switches, and other hardware. Grouping similar components together and using clear labeling or color-coding techniques
can simplify identification and access. It's also helpful to have a central documentation system to keep track of configurations, IP addresses, and
system changes. By investing time and effort into organizing the physical and virtual aspects of the homelab, users can save valuable time during
troubleshooting, upgrades, and expansions, ensuring a smoother and more efficient homelab experience overall.

<div style="float: left; width: 230px; background: white;"><img src="/static/homelab/00_intro/10_reality_cabling.jpg" alt="" style="border:15px solid #FFF"></div>
Cabling is a crucial aspect of homelab organization, and a few extra tips can help maintain a tidy setup. Planning the cabling layout in advance,
considering cable lengths and future expansions, reduces clutter. Utilizing color-coded cables or cable sleeves simplifies identification. Within server
racks or cabinets, employing cable management tools like arms, managers, and routing channels guides cables neatly and improves airflow. Regular cable
audits and cleanups remove unused cables and ensure secure connections. Documenting the cabling infrastructure with diagrams or management software aids
in troubleshooting and modifications, preventing confusion and saving time. By implementing these strategies, the homelab maintains an organized and
efficient cabling system.

## Before

Building a homelab is all about trial and error, and things can get a bit messy and disorganized at times. It's totally normal!
When you start setting up your homelab, you might run into issues and face configuration problems. But don't worry, that's how you
learn! Embrace the challenges and see them as opportunities to grow your skills. Your homelab might end up looking like a tangled
web of cables, but that's part of the fun.

{% assign galleryImages = "/static/homelab/02_building/before/00.jpg|/static/homelab/02_building/before/01.jpg|/static/homelab/02_building/before/02.jpg|/static/homelab/02_building/before/03.jpg|/static/homelab/02_building/before/04.jpg|/static/homelab/02_building/before/05.jpg|/static/homelab/02_building/before/06.jpg|/static/homelab/02_building/before/07.jpg" | split: "|" %}
{% assign galleryTitle = "Before homelab gallery" %}
{% include block_gallery.html galleryTitle=galleryTitle galleryImages=galleryImages %}

Take the time to label and document everything-it'll save you headaches down the road.
And remember, there's a whole [homelab](https://www.reddit.com/r/homelab) community out there
ready to help you out when things get rough!

## BoM

Next are the details of my enclosure homelab's bill of materials (BoM).
Picture this, a comprehensive list of all the hardware and equipment required to build this box of awesomeness.
The BoM is like a treasure map, guiding me through the vast landscape of parts and stuff
to deep dive into the heart and soul of my homelab setup.
There are many parts not in this BoM list because they are not used in the final layout.

| Item                                       | Quantity | Price | Total |Image|
|--------------------------------------------|----------|-------|-------|-----|
| Countertop (240cm, cut in 2x70cm & 2x50cm) | 1        | TBD   | TBD   | <img src="/static/homelab/01_materials/01_tablero.jpg" width="50" height="50" > |
| T-slot 2020 profiles                       | 12       | TBD   | TBD   | <img src="/static/homelab/01_materials/02_t-slot.jpg" width="50" height="50" > |
| 3 way connectors                           | 8        | TBD   | TBD   | <img src="/static/homelab/01_materials/03_3_way_connector.jpg" width="50" height="50" > |
| M4 hexagonal head screws (5mm and 25mm)    | 50/50    | TBD   | TBD   | <img src="/static/homelab/01_materials/04_m3_4mmlong_screw.jpg" width="50" height="50" > |
| T-slot 2020 corner connectors              | 8        | TBD   | TBD   | <img src="/static/homelab/01_materials/06_90_degree_connector.jpg" width="50" height="50" > |
| T-slot 90 degrees connectors               | 50       | TBD   | TBD   | <img src="/static/homelab/01_materials/07_90_degree_connector.jpg" width="50" height="50" > |
| T-slot M4 nut                              | 50       | TBD   | TBD   | <img src="/static/homelab/01_materials/08_t-slot_nut.jpg" width="50" height="50" > |
| Spax wood screws 2mmx25mm                  | 50       | TBD   | TBD   | <img src="/static/homelab/01_materials/09_spax_2mm_screw.jpg" width="50" height="50" > |
| L (90 degrees) bracket 30mmx80mmx55mmx2mm  | 2        | TBD   | TBD   | <img src="/static/homelab/01_materials/12_L_bracket_30X80X55X2.5MM_SIMPSON.jpg" width="50" height="50" > |
| Heavy 90 degrees connectors                | 50       | TBD   | TBD   | <img src="/static/homelab/01_materials/13_corner.jpg" width="50" height="50" > |
| Heavy floor feets                          | 4        | TBD   | TBD   | <img src="/static/homelab/01_materials/16_feet.jpg" width="50" height="50" > |
| Upper door shock absorver                  | 1        | TBD   | TBD   | <img src="/static/homelab/01_materials/17_shock.jpg" width="50" height="50" > |
| Nylon wheels (1 inch), M8                  | 4        | TBD   | TBD   | <img src="/static/homelab/01_materials/18_wheels_1inch_nylon.jpg" width="50" height="50" > |
| M8 hexagon connector                       | 4        | TBD   | TBD   | <img src="/static/homelab/01_materials/19_m10_hexagon_connector.png" width="50" height="50" > |
| Washers                                    | 50       | TBD   | TBD   | <img src="/static/homelab/01_materials/21_washers.jpg" width="50" height="50" > |
| Hinges                                     | 2        | TBD   | TBD   | <img src="/static/homelab/01_materials/22_hinge.jpg" width="50" height="50" > |
| Glass doors (255mmx610mm)                  | 2        | TBD   | TBD   | <img src="/static/homelab/01_materials/23_glass_doors_5mm.jpg" width="50" height="50" > |
| 19" rack profile  (2x12U,4x2U,2x4U)        | 8        | TBD   | TBD   | <img src="/static/homelab/01_materials/81_Adam_Hall_19inch_61535.jpg" width="50" height="50" > |
| 19" rack shelf (250mm)                     | 1        | TBD   | TBD   | <img src="/static/homelab/01_materials/91_shelve.jpg" width="50" height="50" > |
| 19" rack shelf (150mm)                     | 1        | TBD   | TBD   | <img src="/static/homelab/01_materials/92_shelve_150mm.jpg" width="50" height="50" > |
| 19" rack power distributor                 | 1        | TBD   | TBD   | <img src="/static/homelab/01_materials/93_power.jpg" width="50" height="50" > |
| 19" rack rail depth adapter kit            | 2        | TBD   | TBD   | <img src="/static/homelab/01_materials/95_RDA2U.jpg" width="50" height="50" > |

{% assign galleryImages = "/static/homelab/01_materials/01_tablero.jpg|/static/homelab/01_materials/02_t-slot.jpg|/static/homelab/01_materials/03_3_way_connector.jpg|/static/homelab/01_materials/04_m3_4mmlong_screw.jpg|/static/homelab/01_materials/06_90_degree_connector.jpg|/static/homelab/01_materials/07_90_degree_connector.jpg|/static/homelab/01_materials/08_t-slot_nut.jpg|/static/homelab/01_materials/09_spax_2mm_screw.jpg|/static/homelab/01_materials/12_L_bracket_30X80X55X2.5MM_SIMPSON.jpg|/static/homelab/01_materials/13_corner.jpg|/static/homelab/01_materials/16_feet.jpg|/static/homelab/01_materials/17_shock.jpg|/static/homelab/01_materials/18_wheels_1inch_nylon.jpg|/static/homelab/01_materials/19_m10_hexagon_connector.png|/static/homelab/01_materials/21_washers.jpg|/static/homelab/01_materials/22_hinge.jpg|/static/homelab/01_materials/23_glass_doors_5mm.jpg|/static/homelab/01_materials/81_Adam_Hall_19inch_61535.jpg|/static/homelab/01_materials/91_shelve.jpg|/static/homelab/01_materials/92_shelve_150mm.jpg|/static/homelab/01_materials/93_power.jpg|/static/homelab/01_materials/95_RDA2U.jpg" | split: "|" %}
{% assign galleryTitle = "Materials gallery" %}
{% include block_gallery.html galleryTitle=galleryTitle galleryImages=galleryImages %}

In conclusion, the bill of materials (BoM) serves as the backbone of my homelab's main rack enclosure, providing a
detailed roadmap to its inner workings. Through extensive research, careful consideration, and a touch of geeky enthusiasm, I've curated a collection of hardware and equipment that forms the foundation of it.

## After building the homelab

I'll like to share  the journey I embarked on to build my homelab.
It was no walk in the park, but oh , the end result is worth every bit of the sweat and tears.
First, I dove headfirst into the technical realm, researching hardware options like a mad scientist on a caffeine-fueled mission.
Then came the fun part-trial and error galore!. But hey, that's how we learn, right? Finally,
I had my homelab up and running, ready to tackle any tech challenge thrown my way.
So, buckle up, folks, and get ready to witness the fruits of my homelab labor-let the geeky adventures begin!

{% assign galleryImages = "/static/homelab/02_building/after/00.jpg|/static/homelab/02_building/after/01.jpg|/static/homelab/02_building/after/02.jpg|/static/homelab/02_building/after/03.jpg|/static/homelab/02_building/after/04.jpg|/static/homelab/02_building/after/05.jpg|/static/homelab/02_building/after/06.jpg|/static/homelab/02_building/after/07.jpg|/static/homelab/02_building/after/08.jpg|/static/homelab/02_building/after/09.jpg|/static/homelab/02_building/after/10.jpg|/static/homelab/02_building/after/11.jpg|/static/homelab/02_building/after/12.jpg|/static/homelab/02_building/after/13.jpg|/static/homelab/02_building/after/14.jpg|/static/homelab/02_building/after/15.jpg|/static/homelab/02_building/after/16.jpg|/static/homelab/02_building/after/17.jpg|/static/homelab/02_building/after/20.jpg|/static/homelab/02_building/after/21.jpg|/static/homelab/02_building/after/22.jpg|/static/homelab/02_building/after/23.jpg|/static/homelab/02_building/after/24.jpg|/static/homelab/02_building/after/25.jpg|/static/homelab/02_building/after/26.jpg|/static/homelab/02_building/after/27.jpg|/static/homelab/02_building/after/28.jpg|/static/homelab/02_building/after/29.jpg|/static/homelab/02_building/after/30.jpg|/static/homelab/02_building/after/31.jpg|/static/homelab/02_building/after/32.jpg|/static/homelab/02_building/after/33.jpg|/static/homelab/02_building/after/34.jpg|/static/homelab/02_building/after/35.jpg|/static/homelab/02_building/after/36.jpg|/static/homelab/02_building/after/40.jpg|/static/homelab/02_building/after/41.jpg|/static/homelab/02_building/after/42.jpg|/static/homelab/02_building/after/50.jpg|/static/homelab/02_building/after/51.jpg|/static/homelab/02_building/after/52.jpg|/static/homelab/02_building/after/53.jpg|/static/homelab/02_building/after/54.jpg|/static/homelab/02_building/after/55.jpg|/static/homelab/02_building/after/60.jpg|/static/homelab/02_building/after/61.jpg|/static/homelab/02_building/after/80.jpg|/static/homelab/02_building/after/81.jpg|/static/homelab/02_building/after/82.jpg|/static/homelab/02_building/after/83.jpg|/static/homelab/02_building/after/84.jpg|/static/homelab/02_building/after/85.jpg|/static/homelab/02_building/after/86.jpg|/static/homelab/02_building/after/87.jpg|/static/homelab/02_building/after/88.jpg|/static/homelab/02_building/after/89.jpg|/static/homelab/02_building/after/90.jpg|/static/homelab/02_building/after/91.jpg|/static/homelab/02_building/after/92.jpg" | split: "|" %}
{% assign galleryTitle = "After homelab gallery" %}
{% include block_gallery.html galleryTitle=galleryTitle galleryImages=galleryImages %}

My homelab build process was an exhilarating roller coaster ride filled with learning,
challenges, and moments of pure triumph.

## Showcase

After a lot of trial and error, messy cables, I've finally created a tech wonderland right in the comfort of my own home.
Picture this: a sleek rack filled with powerful custom machines, and blinking lights. I've got my own mini data center going on!
With my homelab, I can experiment with cutting-edge technologies, host my own services, and learn like a pro.
It's my own little tech playground, and boy, it feels good to have accomplished this.
Welcome to my homelab, where the possibilities are endless.
BTW all this stuff was 100% sponsored by ME.

{% assign galleryImages = "/static/homelab/03_showcase/00.jpg|/static/homelab/03_showcase/01.jpg|/static/homelab/03_showcase/02.jpg|/static/homelab/03_showcase/03.jpg|/static/homelab/03_showcase/04.jpg|/static/homelab/03_showcase/05.jpg" | split: "|" %}
{% assign galleryTitle = "Showcase gallery" %}
{% include block_gallery.html galleryTitle=galleryTitle galleryImages=galleryImages %}

{% assign videoId = "/static/homelab/02_building/after/93.mp4" %}
{% include rawVideo.html id=videoId %}

{% assign videoId = "/static/homelab/02_building/after/94.mp4" %}
{% include rawVideo.html id=videoId %}

## Deploying

To deploy things here I'm using Ansible, and tools like Kubeinit, these are designed to simplify the
process of setting up and managing complex infrastructure, such as Kubernetes clusters, within a homelab
environment.

Let's talk about deployment tools for homelabs, like Kubeinit. These tools are super handy when you want
to set up and manage complex stuff like Kubernetes clusters in your homelab. Kubeinit, for example, is an
awesome open-source tool that makes deploying Kubernetes a breeze. It takes care of all the nitty-gritty
installation and configuration details, so you don't have to stress about it. Plus, it lets you customize
your cluster to fit your needs. And the best part? There's a friendly community around tools like Kubeinit
that's always ready to lend a hand and share their wisdom.

Kubeinit is an open-source deployment tool specifically tailored for setting up Kubernetes clusters.
It aims to provide an easy and reproducible way to deploy Kubernetes in different configurations,
such as single-node or multi-node clusters. Kubeinit automates the installation process, handles configuration
management, and assists in deploying additional services and tools commonly used with Kubernetes.

Deployment tools like Kubeinit can be highly beneficial for homelab enthusiasts looking to leverage Kubernetes
and containerization technologies. They simplify the setup process, reduce manual work, and provide a standardized
approach to deploying and managing Kubernetes clusters within a homelab environment.

![](/static/homelab/00_intro/deploy.jpg)

## Update log:

<div style="font-size:10px">
  <blockquote>
    <p><strong>2023/05/26:</strong> Initial version.</p>
  </blockquote>
</div>
