---
layout: single-page
title:  "Tech behind this website"
---
This is a static website ([code](https://github.com/adeydas/abhishek.deydas.name)) made with [Jekyll](https://jekyllrb.com/), hosted on [S3](https://aws.amazon.com/s3/) and fronted by [CloudFront](https://aws.amazon.com/cloudfront/).

It also uses [AWS CodeBuild](https://aws.amazon.com/codebuild/) to continuously deploy changes on git push. [Here are the build commands](https://github.com/adeydas/abhishek.deydas.name/blob/master/buildspec.yml) behind the magic.