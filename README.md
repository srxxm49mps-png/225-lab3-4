# 225-lab3-4
CI/CD pipeline Example.  Deploys to DEV with a __ClusterIP and Ingress__, then to PROD with a __LoadBalanced IP__.  Assumes that you have run the MetalLB-Deploy Pipeline at https://github.com/miamioh-cit/metallb-deploy.  HTML is linted.

1) Create a new blank repository in your GitHub account using the lab name above.
2) Copy the URL of this repository and paste it into your repository.
3) View each file, and make changes where it is commented.
4) Run your pipeline.
5) Demonstrate your changes to the code in your video, and show the resulting web page.

## This deployment

DEV: http://10.48.229.166/lab3-4/

PROD: http://10.48.229.166:8080/

This single-node CET cluster already uses port 80 for ingress. The production
LoadBalancer service uses port 8080 on the same existing node IP and forwards
to NGINX port 80 inside the production pod. DEV and PROD have separate selectors.
The course-specified MetalLB v0.14.3 is installed in `metallb-system`.
`metallb-production.yaml` configures only this cluster's own IP as `/32`, with
auto-assignment disabled, instead of exposing the entire shared `/24` subnet.
The existing IP pool was validated with a server-side dry run before use.
This is specific to the current single-node setup and pinned MetalLB version.

The pipeline checks that it is operating on the expected `minterjt-fl26` node,
lints the HTML, builds/pushes the image, deploys both environments, waits for
production IP assignment, and verifies the production page before reporting success.
