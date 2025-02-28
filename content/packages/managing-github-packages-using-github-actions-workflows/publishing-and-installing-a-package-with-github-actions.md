---
title: Publishing and installing a package with GitHub Actions
intro: 'You can configure a workflow in {% data variables.product.prodname_actions %} to automatically publish or install a package from {% data variables.product.prodname_registry %}.'
product: '{% data reusables.gated-features.packages %}'
redirect_from:
  - /github/managing-packages-with-github-packages/using-github-packages-with-github-actions
  - /packages/using-github-packages-with-your-projects-ecosystem/using-github-packages-with-github-actions
  - /packages/guides/using-github-packages-with-github-actions
versions:
  fpt: '*'
  ghes: '*'
  ghae: '*'
  ghec: '*'
shortTitle: Publish & install with Actions
---

{% data reusables.package_registry.packages-ghes-release-stage %}
{% data reusables.package_registry.packages-ghae-release-stage %}

## About {% data variables.product.prodname_registry %} with {% data variables.product.prodname_actions %}

{% data reusables.repositories.about-github-actions %} {% data reusables.repositories.actions-ci-cd %} For more information, see "[AUTOTITLE](/actions/learn-github-actions)."

You can extend the CI and CD capabilities of your repository by publishing or installing packages as part of your workflow.

{% ifversion packages-registries-v2 %}
### Authenticating to package registries with granular permissions

Some {% data variables.product.prodname_registry %} registries support granular permissions. This means you can choose to allow packages to be scoped to a user or an organization, or linked to a repository. For the list of registries that support granular permissions, see "[AUTOTITLE](/packages/learn-github-packages/about-permissions-for-github-packages#granular-permissions-for-userorganization-scoped-packages)."

{% data reusables.package_registry.authenticate_with_pat_for_v2_registry %}

### Authenticating to package registries with repository-scoped permissions

{% endif %}

{% ifversion packages-registries-v2 %}Some {% data variables.product.prodname_registry %} registries only support repository-scoped permissions, and do not support granular permissions. For a list of these registries, see "[AUTOTITLE](/packages/learn-github-packages/about-permissions-for-github-packages#permissions-for-repository-scoped-packages)."

If you want your workflow to access a {% data variables.product.prodname_registry %} registry that does not support granular permissions, then{% else %}To authenticate to package registries on {% data variables.product.product_name %},{% endif %} we recommend using the `GITHUB_TOKEN` that {% data variables.product.product_name %} automatically creates for your repository when you enable {% data variables.product.prodname_actions %}. You should set the permissions for this access token in the workflow file to grant read access for the `contents` scope and write access for the `packages` scope. For forks, the `GITHUB_TOKEN` is granted read access for the parent repository. For more information, see "[AUTOTITLE](/actions/security-guides/automatic-token-authentication)."

You can reference the `GITHUB_TOKEN` in your workflow file using the {% raw %}`{{secrets.GITHUB_TOKEN}}`{% endraw %} context. For more information, see "[AUTOTITLE](/actions/security-guides/automatic-token-authentication)."

## About permissions and package access

{% ifversion packages-registries-v2 %}

### Packages scoped to users or organizations

Registries that support granular permissions allow users to create and administer packages as free-standing resources at the organization level. Packages can be scoped to an organization or personal account and you can customize access to each of your packages separately from repository permissions.

All workflows accessing registries that support granular permissions should use the `GITHUB_TOKEN` instead of a {% data variables.product.pat_generic %}. For more information about security best practices, see "[AUTOTITLE](/actions/security-guides/security-hardening-for-github-actions#using-secrets)."

### Packages scoped to repositories

{% endif %}

When you enable GitHub Actions, GitHub installs a GitHub App on your repository. The `GITHUB_TOKEN` secret is a GitHub App installation access token. You can use the installation access token to authenticate on behalf of the GitHub App installed on your repository. The token's permissions are limited to the repository that contains your workflow. For more information, see "[AUTOTITLE](/actions/security-guides/automatic-token-authentication#about-the-github_token-secret)."

{% data variables.product.prodname_registry %} allows you to push and pull packages through the `GITHUB_TOKEN` available to a {% data variables.product.prodname_actions %} workflow.

{% ifversion packages-registries-v2 %}

## Default permissions and access settings for packages modified through workflows

For packages in registries that support granular permissions, when you create, install, modify, or delete a package through a workflow, there are some default permission and access settings used to ensure admins have access to the workflow. You can adjust these access settings as well. For the list of registries that support granular permissions, see "[AUTOTITLE](/packages/learn-github-packages/about-permissions-for-github-packages#granular-permissions-for-userorganization-scoped-packages)."

For example, by default if a workflow creates a package using the `GITHUB_TOKEN`, then:
- The package inherits the visibility and permissions model of the repository where the workflow is run.
- Repository admins where the workflow is run become the admins of the package once the package is created.

These are more examples of how default permissions work for workflows that manage packages.

| {% data variables.product.prodname_actions %} workflow task | Default permissions and access |
|----|----|
| Download an existing  | - If the package is public, any workflow running in any repository can download the package. <br> - If the package is internal, then all workflows running in any repository owned by the Enterprise account can download the package. For enterprise-owned organizations, you can read any repository in the enterprise <br> - If the package is private, only workflows running in repositories that are given read permission on that package can download the package. <br>
| Upload a new version to an existing package | - If the package is private, internal, or public, only workflows running in repositories that are given write permission on that package can upload new versions to the package.
| Delete a package or versions of a package | - If the package is private, internal, or public, only workflows running in repositories that are given admin permission can delete existing versions of the package.

You can also adjust access to packages in a more granular way or adjust some of the default permissions behavior. For more information, see "[AUTOTITLE](/packages/learn-github-packages/configuring-a-packages-access-control-and-visibility)."

{% endif %}

## Publishing a package using an action

You can use {% data variables.product.prodname_actions %} to automatically publish packages as part of your continuous integration (CI) flow. This approach to continuous deployment (CD) allows you to automate the creation of new package versions, if the code meets your quality standards. For example, you could create a workflow that runs CI tests every time a developer pushes code to a particular branch. If the tests pass, the workflow can publish a new package version to {% data variables.product.prodname_registry %}.

{% data reusables.package_registry.actions-configuration %}

The following example demonstrates how you can use {% data variables.product.prodname_actions %} to build {% ifversion not fpt or ghec %}and test{% endif %} your app, and then automatically create a Docker image and publish it to {% data variables.product.prodname_registry %}.

Create a new workflow file in your repository (such as `.github/workflows/deploy-image.yml`), and add the following YAML:

{% ifversion fpt or ghec %}
{% data reusables.package_registry.publish-docker-image %}

{% else %}

```yaml copy
{% data reusables.actions.actions-not-certified-by-github-comment %}

{% data reusables.actions.actions-use-sha-pinning-comment %}

name: Create and publish a Docker image

on:
  push:
    branches: ['release']

jobs:
  run-npm-build:
    runs-on: ubuntu-latest
    steps:
      - uses: {% data reusables.actions.action-checkout %}
      - name: npm install and build webpack
        run: |
          npm install
          npm run build
      - uses: {% data reusables.actions.action-upload-artifact %}
        with:
          name: webpack artifacts
          path: public/

  run-npm-test:
    runs-on: ubuntu-latest
    needs: run-npm-build
    strategy:
      matrix:
        os: [ubuntu-latest]
        node-version: [12.x, 14.x]
    steps:
      - uses: {% data reusables.actions.action-checkout %}
      - name: Use Node.js {% raw %}${{ matrix.node-version }}{% endraw %}
        uses: {% data reusables.actions.action-setup-node %}
        with:
          node-version: {% raw %}${{ matrix.node-version }}{% endraw %}
      - uses: {% data reusables.actions.action-download-artifact %}
        with:
          name: webpack artifacts
          path: public
      - name: npm install, and test
        run: |
          npm install
          npm test
        env:
          CI: true

  build-and-push-image:
    runs-on: ubuntu-latest
    needs: run-npm-test {% ifversion ghes or ghae %}
    permissions:
      contents: read
      packages: write {% endif %}
    steps:
      - name: Checkout
        uses: {% data reusables.actions.action-checkout %}
      - name: Log in to GitHub Docker Registry
        uses: docker/login-action@65b78e6e13532edd9afa3aa52ac7964289d1a9c1
        with:
          registry: {% ifversion ghae %}docker.YOUR-HOSTNAME.com{% else %}docker.pkg.github.com{% endif %}
          username: {% raw %}${{ github.actor }}{% endraw %}
          password: {% raw %}${{ secrets.GITHUB_TOKEN }}{% endraw %}
      - name: Build and push Docker image
        uses: docker/build-push-action@f2a1d5e99d037542a71f64918e516c093c6f3fc4
        with:
          push: true
          tags: |
            {% ifversion ghae %}docker.YOUR-HOSTNAME.com{% else %}docker.pkg.github.com{% endif %}/{% raw %}${{ github.repository }}/octo-image:${{ github.sha }}{% endraw %}
```

{% endif %}

The relevant settings are explained in the following table. For full details about each element in a workflow, see "[AUTOTITLE](/actions/using-workflows/workflow-syntax-for-github-actions)."

<table>
<tr>
  <th scope="col">Code</th>
  <th scope="col">Explanation</th>
</tr>
<tr>
<td>
{% raw %}
```yaml
on:
  push:
    branches: ['release']
```
{% endraw %}
</td>
<td>
  Configures the <code>Create and publish a Docker image</code> workflow to run every time a change is pushed to the branch called <code>release</code>.
</td>
</tr>

{% ifversion fpt or ghec %}

<tr>
<td>
{% raw %}
```yaml
env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}
```
{% endraw %}
</td>
<td>
  Defines two custom environment variables for the workflow. These are used for the {% data variables.product.prodname_container_registry %} domain, and a name for the Docker image that this workflow builds.
</td>
</tr>

<tr>
<td>
{% raw %}
```yaml
jobs:
  build-and-push-image:
    runs-on: ubuntu-latest
```
{% endraw %}
</td>
<td>
  There is a single job in this workflow. It's configured to run on the latest available version of Ubuntu.
</td>
</tr>

{% else %}

<tr>
<td>

```yaml
run-npm-build:
  runs-on: ubuntu-latest
  steps:
    - uses: {% data reusables.actions.action-checkout %}
    - name: npm install and build webpack
      run: |
        npm install
        npm run build
    - uses: {% data reusables.actions.action-upload-artifact %}
      with:
        name: webpack artifacts
        path: public/
```

</td>
<td>
  This job installs npm and uses it to build the app.
</td>
</tr>

<tr>
<td>

```yaml
run-npm-test:
  runs-on: ubuntu-latest
  needs: run-npm-build
  strategy:
    matrix:
      os: [ubuntu-latest]
      node-version: [12.x, 14.x]
  steps:
    - uses: {% data reusables.actions.action-checkout %}
    - name: Use Node.js {% raw %}${{ matrix.node-version }}{% endraw %}
      uses: {% data reusables.actions.action-setup-node %}
      with:
        node-version: {% raw %}${{ matrix.node-version }}{% endraw %}
    - uses: {% data reusables.actions.action-download-artifact %}
      with:
        name: webpack artifacts
        path: public
    - name: npm install, and test
      run: |
        npm install
        npm test
      env:
        CI: true
```

</td>
<td>
  This job uses <code>npm test</code> to test the code. The <code>needs: run-npm-build</code> command makes this job dependent on the <code>run-npm-build</code> job.
</td>
</tr>

<tr>
<td>
{% raw %}
```yaml
build-and-push-image:
  runs-on: ubuntu-latest
  needs: run-npm-test
```
{% endraw %}
</td>
<td>
  This job publishes the package. The <code>needs: run-npm-test</code> command makes this job dependent on the <code>run-npm-test</code> job.
</td>
</tr>

{% endif %}

<tr>
<td>
{% raw %}
```yaml
permissions:
  contents: read
  packages: write
```
{% endraw %}
</td>
<td>
  Sets the permissions granted to the <code>GITHUB_TOKEN</code> for the actions in this job.
</td>
</tr>

{% ifversion fpt or ghec %}
<tr>
<td>
{% raw %}
```yaml
- name: Log in to the Container registry
  uses: docker/login-action@65b78e6e13532edd9afa3aa52ac7964289d1a9c1
  with:
    registry: ${{ env.REGISTRY }}
    username: ${{ github.actor }}
    password: ${{ secrets.GITHUB_TOKEN }}
```
{% endraw %}
</td>
<td>
  Creates a step called <code>Log in to the {% data variables.product.prodname_container_registry %}</code>, which logs in to the registry using the account and password that will publish the packages. Once published, the packages are scoped to the account defined here.
</td>
</tr>

<tr>
<td>
{% raw %}
```yaml
- name: Extract metadata (tags, labels) for Docker
  id: meta
  uses: docker/metadata-action@9ec57ed1fcdbf14dcef7dfbe97b2010124a938b7
  with:
    images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
```
{% endraw %}
</td>
<td>
  This step uses <code><a href="https://github.com/docker/metadata-action#about">docker/metadata-action</a></code> to extract tags and labels that will be applied to the specified image. The <code>id</code> "meta" allows the output of this step to be referenced in a subsequent step. The <code>images</code> value provides the base name for the tags and labels.
</td>
</tr>

{% else %}
<tr>
<td>
{% raw %}
```yaml
- name: Log in to GitHub Docker Registry
  uses: docker/login-action@65b78e6e13532edd9afa3aa52ac7964289d1a9c1
  with:
    registry: {% endraw %}{% ifversion ghae %}docker.YOUR-HOSTNAME.com{% else %}docker.pkg.github.com{% endif %}{% raw %}
    username: ${{ github.actor }}
    password: ${{ secrets.GITHUB_TOKEN }}
```
{% endraw %}
</td>
<td>
  Creates a new step called <code>Log in to GitHub Docker Registry</code>, which logs in to the registry using the account and password that will publish the packages. Once published, the packages are scoped to the account defined here.
</td>
</tr>
{% endif %}

<tr>
<td>
{% raw %}
```yaml
- name: Build and push Docker image
```
{% endraw %}
</td>
<td>
  Creates a new step called <code>Build and push Docker image</code>. This step runs as part of the <code>build-and-push-image</code> job.
</td>
</tr>

<tr>
<td>
{% raw %}
```yaml
uses: docker/build-push-action@f2a1d5e99d037542a71f64918e516c093c6f3fc4
```
{% endraw %}
</td>
<td>
  Uses the Docker <code>build-push-action</code> action to build the image, based on your repository's <code>Dockerfile</code>. If the build succeeds, it pushes the image to {% data variables.product.prodname_registry %}.
</td>
</tr>

<tr>
<td>
{% raw %}
```yaml
with:
```
{% endraw %}
</td>
<td>
  Sends the required parameters to the <code>build-push-action</code> action. These are defined in the subsequent lines.
</td>
</tr>

{% ifversion fpt or ghec %}
<tr>
<td>
{% raw %}
```yaml
context: .
```
{% endraw %}
</td>
<td>
  Defines the build's context as the set of files located in the specified path. For more information, see "<a href="https://github.com/docker/build-push-action#usage">Usage</a>."
</td>
</tr>
{% endif %}

<tr>
<td>
{% raw %}
```yaml
push: true
```
{% endraw %}
</td>
<td>
  Pushes this image to the registry if it is built successfully.
</td>
</tr>

{% ifversion fpt or ghec %}
<tr>
<td>
{% raw %}
```yaml
tags: ${{ steps.meta.outputs.tags }}
labels: ${{ steps.meta.outputs.labels }}
```
{% endraw %}
</td>
<td>
  Adds the tags and labels extracted in the "meta" step.
</td>
</tr>

{% else %}
<tr>
<td>
{% ifversion ghae %}
{% raw %}
```yaml
tags: |
docker.YOUR-HOSTNAME.com/${{ github.repository }}/octo-image:${{ github.sha }}
```
{% endraw %}
{% else %}
{% raw %}
```yaml
tags: |
docker.pkg.github.com/${{ github.repository }}/octo-image:${{ github.sha }}
```
{% endraw %}
{% endif %}
</td>
<td>
  Tags the image with the SHA of the commit that triggered the workflow.
</td>
</tr>
{% endif %}

</table>

This new workflow will run automatically every time you push a change to a branch named `release` in the repository. You can view the progress in the **Actions** tab.

A few minutes after the workflow has completed, the new package will visible in your repository. To find your available packages, see "[AUTOTITLE](/packages/learn-github-packages/viewing-packages#viewing-a-repositorys-packages)."

## Installing a package using an action

You can install packages as part of your CI flow using {% data variables.product.prodname_actions %}. For example, you could configure a workflow so that anytime a developer pushes code to a pull request, the workflow resolves dependencies by downloading and installing packages hosted by {% data variables.product.prodname_registry %}. Then, the workflow can run CI tests that require the dependencies.

Installing packages hosted by {% data variables.product.prodname_registry %} through {% data variables.product.prodname_actions %} requires minimal configuration or additional authentication when you use the `GITHUB_TOKEN`.{% ifversion fpt or ghec %} Data transfer is also free when an action installs a package. For more information, see "[AUTOTITLE](/billing/managing-billing-for-github-packages/about-billing-for-github-packages)."{% endif %}

{% data reusables.package_registry.actions-configuration %}

{% ifversion packages-registries-v2 %}
## Upgrading a workflow that accesses a registry using a {% data variables.product.pat_generic %}

{% data variables.product.prodname_registry %} supports the `GITHUB_TOKEN` for easy and secure authentication in your workflows. If you're using a registry that supports granular permissions, and your workflow is using a {% data variables.product.pat_generic %} to authenticate to the registry, then we highly recommend you update your workflow to use the `GITHUB_TOKEN`.

For more information about the `GITHUB_TOKEN`, see "[AUTOTITLE](/actions/security-guides/automatic-token-authentication#using-the-github_token-in-a-workflow)."

Using the `GITHUB_TOKEN`, instead of a {% data variables.product.pat_v1 %} with the `repo` scope, increases the security of your repository as you don't need to use a long-lived {% data variables.product.pat_generic %} that offers unnecessary access to the repository where your workflow is run. For more information about security best practices, see "[AUTOTITLE](/actions/security-guides/security-hardening-for-github-actions#using-secrets)."

1. Navigate to your package landing page.
{% data reusables.package_registry.package-settings-actions-access %}
1. To ensure your package has access to your workflow, you must add the repository where the workflow is stored to your package. {% data reusables.package_registry.package-settings-add-repo %}
   {% note %}

   **Note:** Adding a repository to your package {% data variables.package_registry.package-settings-actions-access-menu %} is different than connecting your package to a repository. For more information, see "[AUTOTITLE](/packages/learn-github-packages/configuring-a-packages-access-control-and-visibility#ensuring-workflow-access-to-your-package)" and "[AUTOTITLE](/packages/learn-github-packages/connecting-a-repository-to-a-package)."

   {% endnote %}
1. Optionally, use {% data variables.package_registry.package-settings-actions-access-role-dropdown %}
1. Open your workflow file. On the line where you log in to the registry, replace your {% data variables.product.pat_generic %} with {% raw %}`${{ secrets.GITHUB_TOKEN }}`{% endraw %}.

For example, this workflow publishes a Docker image to the {% data variables.product.prodname_container_registry %} and uses {% raw %}`${{ secrets.GITHUB_TOKEN }}`{% endraw %} to authenticate.

```yaml copy
name: Demo Push

on:
  push:
    # Publish `main` as Docker `latest` image.
    branches:
      - main
      - seed

    # Publish `v1.2.3` tags as releases.
    tags:
      - v*

  # Run tests for any PRs.
  pull_request:

env:
  IMAGE_NAME: ghtoken_product_demo

jobs:
  # Push image to GitHub Packages.
  # See also https://docs.docker.com/docker-hub/builds/
  push:
    runs-on: ubuntu-latest
    permissions:
      packages: write
      contents: read

    steps:
      - uses: {% data reusables.actions.action-checkout %}

      - name: Build image
        run: docker build . --file Dockerfile --tag $IMAGE_NAME --label "runnumber=${GITHUB_RUN_ID}"

      - name: Log in to registry
        # This is where you will update the {% data variables.product.pat_generic %} to GITHUB_TOKEN
        run: echo "{% raw %}${{ secrets.GITHUB_TOKEN }}{% endraw %}" | docker login ghcr.io -u ${{ github.actor }} --password-stdin

      - name: Push image
        run: |
          IMAGE_ID=ghcr.io/{% raw %}${{ github.repository_owner }}{% endraw %}/$IMAGE_NAME

          # Change all uppercase to lowercase
          IMAGE_ID=$(echo $IMAGE_ID | tr '[A-Z]' '[a-z]')
          # Strip git ref prefix from version
          VERSION=$(echo "{% raw %}${{ github.ref }}{% endraw %}" | sed -e 's,.*/\(.*\),\1,')
          # Strip "v" prefix from tag name
          [[ "{% raw %}${{ github.ref }}{% endraw %}" == "refs/tags/"* ]] && VERSION=$(echo $VERSION | sed -e 's/^v//')
          # Use Docker `latest` tag convention
          [ "$VERSION" == "main" ] && VERSION=latest
          echo IMAGE_ID=$IMAGE_ID
          echo VERSION=$VERSION
          docker tag $IMAGE_NAME $IMAGE_ID:$VERSION
          docker push $IMAGE_ID:$VERSION
```

{% endif %}
