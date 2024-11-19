---
title: ISC Student Cluster Competition 2024
summary: "UPDATE: We ranked 8th out of 21 teams worldwide. Pretty good for the only first time team in the competition!"
tags:
  - HPC
  - ISC SCC
date: 2024-04-01T00:00:00+05:30

# Optional external URL for project (replaces project detail page).
external_link: ''

image:
  caption: The IIT Kanpur Team at the 2024 ISC Student Cluster Competition
  focal_point: Smart

links:
  - icon: twitter
    icon_pack: fab
    name: Follow
    url: https://twitter.com/ExaDecimals
url_code: ''
url_pdf: ''
url_slides: 'https://iitk-my.sharepoint.com/:p:/g/personal/spratham21_iitk_ac_in/ES0TYculjr5LvFX3cIbL9BEBcbJFCOnXQwmbV857FBHeaw'
url_video: ''

# Slides (optional).
#   Associate this project with Markdown slides.
#   Simply enter your slide deck's filename without extension.
#   E.g. `slides = "example-slides"` references `content/slides/example-slides.md`.
#   Otherwise, set `slides = ""`.
#slides: example
---

The ISC Student Cluster Competition (SCC) is an annual supercomputing competition that focuses on the HPC community's future generation. 
The IIT Kanpur team. 'ExaDecimals', is a group of third year undergraduate students from the Department of Computer Science and Engineering. The team is mentored by Dr. Preeti Malakar and Dr. Swarnendu Biswas, who are very knowledgeable in the field of HPC and have guided the team through the competition. 

<!-- Insert image -->
![ISC SCC 2024](images/teams.jpeg)

Our team was selected to participate in the 2024 ISC Student Cluster Competition, in the virtual round. We submitted our proposal in October last year and became one of the first Indian team to participate in the competition, and also one of the very few first time participants in this years event. We were competing agaisnt top universities from all over the world, and were aware of our lack of experience in the field. However, we were determined to learn and make the most of this opportunity.

## Preparation Phase
Our early preperations started with learning about building, running, profiling and visualizing applications in the local CSE beouwulf cluster, then followed by Param Sanganak, our in house Supercomputer at IITK. We initially started with trying out the tasks given in the previous years of the competition.
## Competition Overview

The competition was 2 months long and began in March. We were given 4 tasks to complete which covered a wide range of topics, including parallelization, optimizations, scaling, profiling and visualization.

### Tasks Overview
<!-- List of tasks -->
1. **Parallelization and Optimization of ICON Model:**
The first task was to parallelize and optimize the standalone microphysics module of the ICON model. We were alloted OpenACC to parallelize the code and make it compatible with the NVIDIA GPUs. We had to optimize the code to run on the DKRZ Supercomputer, which is rank 60 in the Top500 list. Our initial efforts were just to run the code on GPU and get the output. We then started optimizing the code by using the OpenACC directives and by performing loop reorderings, breaking loops, etc. We were able to achieve a speedup of 20x over the CPU version of the code, and maintaining correctness.

2. **Profiling and Optimization of RegCM:**
The second task was to run, profile and visualize the runs of RegCM for the given dataset. We were asked to run this on the Pittsburgh Supercomputing Center's Bridges-2 Supercomputer. We also attempted the bonus task of optimizing the microphysics module of the RegCM model. We obtained a speedup of 23% over the baseline code and submitted a pull request to the RegCM repository. This bonus task was completed by a very few teams and ours was the best among the visible pull requests by a very large margin.
3. **Analysis and Optimization of Conquest Application:**
The third task was to analyze and find the best configuration of OpenMP threads and MPI processes for the Conquest application. We were also asked to profile the application. Our team also performed weak scaling and strong scaling experiments on the application.

4. **Profiling and Visualization of NekoCFD:**
The final task was to run, profile and visualise the NekoCFD code with CPU and GPU backends. This was also run on the Bridges-2 Supercomputer. We ran the code for different configurations, profiled using Tau and visualized the output files using Paraview.


Overall the competition was a great learning experience for us. We learned a lot about HPC, parallelization, optimizations, profiling and visualization. We also learned about the importance of teamwork, communication and time management. We were able to complete all the tasks and submit the final report on time. We are eagerly waiting for the results of the competition and hope to participate in the next edition of the competition.

**Update: We ranked 8th out of 21 teams worldwide. Pretty good for the only first time team in the competition!**