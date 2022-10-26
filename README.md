# leadership-section-repo

This project contains the clone of the leadership section web page of Decagonhq.com built with simple css and html


version: 2

jobs:
  plan-apply:
    working_directory: /tmp/project
    docker:
      - image: docker.mirror.hashicorp.services/hashicorp/terraform:light
    steps:
      - checkout
      - run:
          name: terraform init & plan
          command: |
            terraform init -backend-config environments/dev/backend.conf -input=false
            terraform plan  -out tfapply -var-file environments/dev/terraform.auto.tfvars
      - persist_to_workspace:
          root: .
          paths:
            - .

  apply:
    docker:
      - image: docker.mirror.hashicorp.services/hashicorp/terraform:light
    steps:
      - attach_workspace:
          at: .
      - run:
          name: terraform
          command: |
            terraform apply -auto-approve tfapply
      - persist_to_workspace:
          root: .
          paths:
            - .

workflows:
  version: 2
  plan_approve_apply:
    jobs:
      - plan-apply
      - hold-apply:
          type: approval
          requires:
            - plan-apply
      - apply:
          requires:
            - hold-apply
 
