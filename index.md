---
layout: category
---

## Continous Integration for an Open Source Project

This website is dedicated to describe a continous integration concept for an Open Source project using Github and Jenkins. Each commit and pull request is being build, the status is visible in Github and can be verified immediatly.

The goal is to have all the files and build logs publicy available but protect the build jobs from corruption. The build jobs are being build in a Docker container to prevent any damage to the Jenkins instance. 

The Jenkins file itself is in the Github repository and is protected from unauthorized change and usage from non trusted users.

```ruby
require 'redcarpet'
markdown = Redcarpet.new("Hello World!")
puts markdown.to_html
```


