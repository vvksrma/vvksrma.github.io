---
# Leave the homepage title empty to use the site title
title: Pratham Sahu
date: 2022-10-24
type: landing

sections:
  
  - block: about.biography
    id: about
    content:
      title: About Me
      # Choose a user profile to display (a folder name within `content/authors/`)
      username: admin
  # - block: features
  #   content:
  #     title: Skills
  #     items:
  #       - name: R
  #         description: 90%
  #         icon: r-project
  #         icon_pack: fab
  #       - name: Statistics
  #         description: 100%
  #         icon: chart-line
  #         icon_pack: fas
  #       - name: Photography
  #         description: 10%
  #         icon: camera-retro
  #         icon_pack: fas
  - block: experience
    content:
      title: Work Experience
      # Date format for experience
      #   Refer to https://wowchemy.com/docs/customization/#date-format
      date_format: Jan 2006
      # Experiences.
      #   Add/remove as many `experience` items below as you like.
      #   Required fields are `title`, `company`, and `date_start`.
      #   Leave `date_end` empty if it's your current employer.
      #   Begin multi-line descriptions with YAML's `|2-` multi-line prefix.
      items:
        - title: Research Intern
          company: Adobe Research, Big Data Experience Lab
          company_url: 'https://research.adobe.com/'
          company_logo: ''
          location: Bengaluru, India
          date_start: '2024-05-13'
          date_end: ''
          description: Systems and Language Research Group

        - title: Intern Research Assistant
          company: Yonsei Vision and learning Laboratories
          company_url: ''
          # company_logo: org-x
          location: Seoul, South Korea
          date_start: '2023-05-01'
          date_end: '2023-08-31'
          description: Performed research on Continual Learning Algorithms

        - title: Platform software engineering intern
          company: mSenseAI
          company_url: 'msense.ai'
          # company_logo: org-gc
          location: Bengaluru
          date_start: '2022-07-01'
          date_end: '2022-08-01'
          description: |2-
              Worked on implementing an platform for client side deployment of product.
        
    design:
      columns: '2'
  
  
  - block: collection
    id: posts
    content:
      title: Recent Posts
      subtitle: ''
      text: ''
      # Choose how many pages you would like to display (0 = all pages)
      count: 5
      # Filter on crit  eria
      filters:
        folders:
          - post
        author: ""
        category: ""
        tag: ""
        exclude_featured: false
        exclude_future: false
        exclude_past: false
        publication_type: ""
      # Choose how many pages you would like to offset by
      offset: 0
      # Page order: descending (desc) or ascending (asc) date.
      order: desc
    design:
      # Choose a layout view
      view: compact
      columns: '2'
  - block: collection
    id: competitions
    content:
      title: Competitions
      subtitle: ''
      text: ''
      # Choose how many pages you would like to display (0 = all pages)
      count: 5
      # Filter on crit  eria
      filters:
        folders:
          - competition
    design:
      # Choose a layout view
      view: compact
      columns: '2' 

  - block: markdown
    content:
      title: Courses
      subtitle: Courses I have taken at IIT Kanpur
      text: |
        | **Course Name**                         | **Course Code** |
        |-----------------------------------------|-------------|
        | Operating Systems                       | CS330       |
        | Introduction to Machine Learning        | CS771       |
        | Software Development and Operations     | CS253       |
        | Probability for Computer Science        | CS203       |
        | Data Structures and Algorithms          | ESO207      |
        | Design and Analysis of Algorithms       | CS345       |
        | Theory of Computation                   | CS340       |
        | Computer Organisation                   | CS220       |
        | Big Data Analytics and Visualization    | CS677       |
        | Linux Kernel Programming                | CS614       |
        | Compiler Design                         | CS335       |
        | Networks                                | CS425       |
        | Parallel Computing                      | CS633       |

    design:
      # See Page Builder docs for all section customization options.
      # Choose how many columns the section has. Valid values: '1' or '2'.
      columns: '2'
  
  - block: markdown
    content:
      title: My Academic Achievements
      text: |

        ### Examinations and Olympiads
        - [**JEE Advanced 2021**](https://jeeadv.ac.in/): Secured an **All India Rank 131** among 1,50,000 shortlisted candidates, organized by IIT Kharagpur.
        - [**JEE Mains 2021**](https://jeemain.nta.nic.in/): Achieved **All India Rank 87** out of 1.1 million candidates, organized by NTA.
        - [**INPhO 2021**](http://olympiads.hbcse.tifr.res.in/subjects/physics):(Gold Medal) Bagged **AIR 44** and progressed to the National Selection Camp for IPhO.
        - [**INAO 2021**](http://olympiads.hbcse.tifr.res.in/subjects/astronomy): Stood with **AIR 31** and made the cut for National Selection Camp for IAO.
        - [**INChO 2021**](http://olympiads.hbcse.tifr.res.in/subjects/chemistry): Clinched **AIR 19** and represented at the National Selection Camp for IChO.
        - [**KVPY SA 2019**](http://www.kvpy.iisc.ernet.in/): Secured **All India Rank 340**, organised by the Indian Institute of Science, Bangalore.
        - [**KVPY SX 2020**](http://www.kvpy.iisc.ernet.in/): Ranked **All India Rank 148**, organised the Indian Institute of Science, Bangalore.
        - [**NTSE 2019**](https://ncert.nic.in/national-talent-examination.php): Proud recipient of the coveted scholarship, an initiative by NCERT.

        ### Awards and Recognitions
        - [**Directors Scholarship**](http://www.iitk.ac.in/): Honored at **IIT Kanpur in 2022** for an outstanding JEE Advanced rank.
        - [**Academic Excellence Award**](http://www.iitk.ac.in/): Recognized consecutively for the academic years 2021-22 and 2022-2023 for exceptional performance.
        - Accepted into the [**Inaugural IEEE-CS TCHPC/TCPP HPC Student Cohort 2024**](https://tc.computer.org/tchpc/home-page/page-of-awards/)


    design:
      columns: '2'
      
  

  # - block: features
  #   content:
  #     title: My Courses at IITK
  #     items:
  #       - name: Operating Systems
  #         description: CS330
  #         icon: 
  #         icon_pack: fas
  #       - name: Introduction to Machine Learning
  #         description: CS771
  #         icon: 
  #         icon_pack: fas
  #       - name: Software Development and Operations
  #         description: CS253
  #         icon: 
  #         icon_pack: fas
  #       - name: Probability for Computer Science
  #         description: CS203
  #         icon: 
  #         icon_pack: fas 
  #       - name: Data Structures and Algorithms
  #         description: ESO207
  #         icon: 
  #         icon_pack: fas 
  #       - name: Design and Analysis of Algorithms
  #         description: CS345
  #         icon: 
  #         icon_pack: fas 
  #       - name: Theory of Computation
  #         description: CS340
  #         icon: 
  #         icon_pack: fas 
  #       - name: Computer Organisation
  #         description: CS220
  #         icon: 
  #         icon_pack: fas
  #       - name: Big Data Analytics and Visualization
  #         description: CS667
  #         icon: 
  #         icon_pack: fas

  # - block: tag_cloud
  #     content:
  #       title: My title
  #       subtitle: My subtitle
  #       text: Add any **markdown** formatted content here - text, images, videos, galleries - and even HTML code!
  #       # Choose a taxonomy from the `taxonomies` list in `config.yaml` to display (e.g. tags, categories, authors)
  #       taxonomy: tags
  #       # Choose how many tags you would like to display (0 = all tags)
  #       count: 20
  #     design:
  #       # Minimum and maximum font sizes (1.0 = 100%).
  #       font_size_min: 0.7
  #       font_size_max: 2.0

  # - block: scholastic
  #   content:
  #     title: Scholastic Achievements
  #     date_format: Jan 2006
  #     items:
  #       - title: Director's Scholarship
  #         organization: IIT Kanpur
  #         date_start: '2022-07-01'
  #         description: Awarded

  #       - title: All India Rank 131
  #         organization: JEE ADVANCED
  #         date_start: '2022-07-01'
  #         description: Secured an All India Rank of 131 among 150k shortlisted canditates

  #       - title: Academic Excellence Award
  #         organization: IIT Kanpur
  #         date_start: '2022-07-01'
  #         description: Recognised for being among the top 10% of the batch.

  #       # Add more scholastic achievements as needed...
  #   design:
  #     columns: '2'

  # - block: accomplishments
  #   content:
  #     # Note: `&shy;` is used to add a 'soft' hyphen in a long heading.
  #     title: 'Academic Achievments'
  #     subtitle:
  #     # Date format: https://wowchemy.com/docs/customization/#date-format
  #     date_format: Jan 2006
  #     # Accomplishments.
  #     #   Add/remove as many `item` blocks below as you like.
  #     #   `title`, `organization`, and `date_start` are the required parameters.
  #     #   Leave other parameters empty if not required.
  #     #   Begin multi-line descriptions with YAML's `|2-` multi-line prefix.
  #     items:
  #       - certificate_url: https://www.coursera.org
  #         date_end: ''
  #         date_start: '2021-01-25'
  #         description: ''
  #         organization: Coursera
  #         organization_url: https://www.coursera.org
  #         title: Neural Networks and Deep Learning
  #         url: ''
  #       - certificate_url: https://www.edx.org
  #         date_end: ''
  #         date_start: '2021-01-01'
  #         description: Formulated informed blockchain models, hypotheses, and use cases.
  #         organization: edX
  #         organization_url: https://www.edx.org
  #         title: Blockchain Fundamentals
  #         url: https://www.edx.org/professional-certificate/uc-berkeleyx-blockchain-fundamentals
  #       - certificate_url: https://www.datacamp.com
  #         date_end: '2020-12-21'
  #         date_start: '2020-07-01'
  #         description: ''
  #         organization: DataCamp
  #         organization_url: https://www.datacamp.com
  #         title: 'Object-Oriented Programming in R'
  #         url: ''
  #   design:
  #     columns: '2'
  
  # - block: portfolio
  #   id: projects
  #   content:
  #     title: Projects
  #     filters:
  #       folders:
  #         - project
  #     # Default filter index (e.g. 0 corresponds to the first `filter_button` instance below).
  #     default_button_index: 0
  #     # Filter toolbar (optional).
  #     # Add or remove as many filters (`filter_button` instances) as you like.
  #     # To show all items, set `tag` to "*".
  #     # To filter by a specific tag, set `tag` to an existing tag name.
  #     # To remove the toolbar, delete the entire `filter_button` block.
  #     buttons:
  #       - name: All
  #         tag: '*'
  #       - name: Deep Learning
  #         tag: Deep Learning
  #       - name: Other
  #         tag: Demo
  #   design:
  #     # Choose how many columns the section has. Valid values: '1' or '2'.
  #     columns: '1'
  #     view: showcase
  #     # For Showcase view, flip alternate rows?
  #     flip_alt_rows: false
  # - block: markdown
  #   content:
  #     title: Gallery
  #     subtitle: ''
  #     text: |-
  #       {{< gallery album="demo" >}}
  #   design:
  #     columns: '1'
  # - block: collection
  #   id: featured
  #   content:
  #     title: Featured Publications
  #     filters:
  #       folders:
  #         - publication
  #       featured_only: true
  #   design:
  #     columns: '2'
  #     view: card
  # - block: collection
  #   content:
  #     title: Recent Publications
  #     text: |-
  #       {{% callout note %}}
  #       Quickly discover relevant content by [filtering publications](./publication/).
  #       {{% /callout %}}
  #     filters:
  #       folders:
  #         - publication
  #       exclude_featured: true
  #   design:
  #     columns: '2'
  #     view: citation
  # - block: collection
  #   id: talks
  #   content:
  #     title: Recent & Upcoming Talks
  #     filters:
  #       folders:
  #         - event
  #   design:
  #     columns: '2'
  #     view: compact
  # - block: tag_cloud
  #   content:
  #     title: Popular Topics
  #   design:
  #     columns: '2'
  # - block: contact
  #   id: contact
  #   content:
  #     title: Contact
  #     subtitle:
  #     text: |-
  #       Lorem ipsum dolor sit amet, consectetur adipiscing elit. Nam mi diam, venenatis ut magna et, vehicula efficitur enim.
  #     # Contact (add or remove contact options as necessary)
  #     email: test@example.org
  #     phone: 888 888 88 88
  #     appointment_url: 'https://calendly.com'
  #     address:
  #       street: 450 Serra Mall
  #       city: Stanford
  #       region: CA
  #       postcode: '94305'
  #       country: United States
  #       country_code: US
  #     directions: Enter Building 1 and take the stairs to Office 200 on Floor 2
  #     office_hours:
  #       - 'Monday 10:00 to 13:00'
  #       - 'Wednesday 09:00 to 10:00'
  #     contact_links:
  #       - icon: twitter
  #         icon_pack: fab
  #         name: DM Me
  #         link: 'https://twitter.com/Twitter'
  #       - icon: skype
  #         icon_pack: fab
  #         name: Skype Me
  #         link: 'skype:echo123?call'
  #       - icon: video
  #         icon_pack: fas
  #         name: Zoom Me
  #         link: 'https://zoom.com'
  #     # Automatically link email and phone or display as text?
  #     autolink: true
  #     # Email form provider
  #     form:
  #       provider: netlify
  #       formspree:
  #         id:
  #       netlify:
  #         # Enable CAPTCHA challenge to reduce spam?
  #         captcha: false
  #   design:
  #     columns: '2'
---
