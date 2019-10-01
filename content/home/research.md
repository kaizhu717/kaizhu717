+++
# A Projects section created with the Portfolio widget.
widget = "portfolio"  # See https://sourcethemes.com/academic/docs/page-builder/
headless = true  # This file represents a page section.
active = true  # Activate this widget? true/false
weight = 30  # Order that this section will appear.

title = "Research"
subtitle = ""

[content]
  # Page type to display. E.g. project.
  page_type = "research"
  
  # Filter toolbar (optional).
  # Add or remove as many filters (`[[content.filter_button]]` instances) as you like.
  # To show all items, set `tag` to "*".
  # To filter by a specific tag, set `tag` to an existing tag name.
  # To remove toolbar, delete/comment all instances of `[[content.filter_button]]` below.
  
  # Default filter index (e.g. 0 corresponds to the first `[[filter_button]]` instance below).
  filter_default = 0
  
  # [[content.filter_button]]
  #   name = "All"
  #   tag = "*"
  
  # [[content.filter_button]]
  #   name = "Deep Learning"
  #   tag = "Deep Learning"
  
  # [[content.filter_button]]
  #   name = "Other"
  #   tag = "Demo"

[design]
  # Choose how many columns the section has. Valid values: 1 or 2.
  columns = "2"

  # Toggle between the various page layout types.
  #   1 = List
  #   2 = Compact
  #   3 = Card
  #   5 = Showcase
  view = 3

  # For Showcase view, flip alternate rows?
  flip_alt_rows = false

[design.background]
  # Apply a background color, gradient, or image.
  #   Uncomment (by removing `#`) an option to apply it.
  #   Choose a light or dark text color by setting `text_color_light`.
  #   Any HTML color name or Hex value is valid.
  
  # Background color.
  # color = "navy"
  
  # Background gradient.
  # gradient_start = "DeepSkyBlue"
  # gradient_end = "SkyBlue"
  
  # Background image.
  # image = "background.jpg"  # Name of image in `static/img/`.
  # image_darken = 0.6  # Darken the image? Range 0-1 where 0 is transparent and 1 is opaque.

  # Text color (true=light or false=dark).
  text_color_light = false 
  
[advanced]
 # Custom CSS. 
 css_style = ""
 
 # CSS class.
 css_class = ""
+++

Content Growth and Attention Contagion in Information Networks: A Natural Experiment on Wikipedia \\
- Kai Zhu, Dylan Walker, Lev Muchnik \\
- Accepted at Information Systems Research \\
- Presented at WWW 2017, CODE 2017, WISE 2017, SCECR 2018, HBS 2018, WEBEIS 2019 \\
- Abstract: Open collaboration platforms have fundamentally changed the way knowledge is produced, disseminated and consumed. In these systems, contributions arise organically with little to no central governance. While such decentralization provides many benefits, a lack of broad oversight and coordination can leave questions of information poverty and skewness to the mercy of the system’s natural dynamics. Unfortunately, we still lack a basic understanding of the dynamics at play in these systems, and specifically, how contribution and attention interact and propagate through information networks. We leverage a large-scale natural experiment to study how exogenous content contributions to Wikipedia articles affect the attention they attract and how that attention spills over to other articles in the network. Results reveal that exogenously added content leads to significant, substantial and long-term increases in both content consumption and subsequent contributions. Furthermore, we find significant attention spillover to downstream hyperlinked articles. Through both analytical estimation and empiricallyinformed simulation, we evaluate policies to harness this attention contagion to address the problem of information poverty and skewness. We find that harnessing attention contagion can lead to as much as a twofold increase in the total attention flow to clusters of disadvantaged articles. Our findings have important policy implications for open collaboration platforms and information networks