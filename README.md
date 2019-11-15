# docker-kubernetes-python-client

Based on https://github.com/edoburu/docker-gitlab-kubernetes-client

This is a small image that allows to run git, docker, kubectl, helm and python3 and pip3.

# Usage

Perform the deployment from .gitlab-ci.yml:
    
    hubhk/gitlab-kubernetes-python-client

    stages:
    - build
    - deploy
    
    before_script:
      # Allow sh substitution to support both tags and commit releases.
      # CI_PIPELINE_ID is useful to force redeploys on pipeline triggers
      # CI_COMMIT_TAG is only filled for tags.
      # CI_COMMIT_REF_NAME can be a tag or branch name
      - export IMAGE_TAG=ci_${CI_PIPELINE_ID}_${CI_COMMIT_TAG:-git_${CI_COMMIT_REF_NAME}_$CI_COMMIT_SHA}
      - pip3 install requests
    
    build image:
      stage: build
      script:
      - docker build -t $CI_REGISTRY_IMAGE:$IMAGE_TAG .
      - docker run --rm $CI_REGISTRY_IMAGE:$IMAGE_TAG /script/to/run/tests
      - docker login -u $CI_REGISTRY_USER -p $CI_JOB_TOKEN $CI_REGISTRY
      - docker push $CI_REGISTRY_IMAGE:$IMAGE_TAG
    
    deploy to production:
      stage: deploy
      environment:
        name: production
        url: http://example.com/
      when: manual
      script:
      - helm upgrade
            --install
            --tiller-namespace="$KUBE_NAMESPACE"
            --namespace "$KUBE_NAMESPACE"
            --reset-values
            --values "values-$CI_ENVIRONMENT_SLUG.yml"
            --set="image.tag=$IMAGE_TAG,nameOverride=$CI_ENVIRONMENT_SLUG"
            "RELEASE_NAME" "CHART_DIR"
      - python3 notify_developers.py
      only:
      - tags
      - triggers
      except:
      - beta  # when part of trigger
      
Install your requirements with `pip3` inside the `before_script`. Launch your scripts with `python3`.  