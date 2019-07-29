---
layout: single-page
title:  "Tech behind this website"
---
This is a static [website](https://github.com/adeydas/abhishek.deydas.name) made with [Jekyll](https://jekyllrb.com/), hosted on [S3](https://aws.amazon.com/s3/) and fronted by [CloudFront](https://aws.amazon.com/cloudfront/).

It also uses [AWS CodeBuild](https://aws.amazon.com/codebuild/) to continuously deploy changes on git push. [Here are the build commands](https://github.com/adeydas/abhishek.deydas.name/blob/master/buildspec.yml) behind the magic.

And all of that infrastructure is managed using [this Cloudformation template](https://github.com/adeydas/S3-Cloudfront-static-website-infrastructure). Typically my workflow for updating the website is a simple git push. Internally it uses the following workflow to build and deploy:
![Website architecture](https://abhishek.deydas.name/images/git_push_workflow.png)