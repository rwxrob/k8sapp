# Kubernetes Applications (K8SAPP) Management Conventions

There are many ways to install and manage Kubernetes applications:

* `kubectl` Scripts
* Kustomize
* Helm
* Modified Helm
* Operator Framework
* Custom Operators
* Adapted Terraform and/or Ansible

Cloud native administrators are forced to learn one or all of these
since there is no agreement in the industry as to any standard.
Moreover, the industry is full of overly-hyped frameworks and "package
managers" vying for dominance while failing to meet even minimal
requirements for air-gapped security and sustainability. This situation,
brought about by attempting to force imperative IT business processes
into declarative "solutions," has created chaos and a climate of
conflict and shame for those refusing to use the Kubernetes application
management "standards" when, in fact, the safest and most sustainable
approach is to keep things simple: just use Kubernetes resource files
and `kubectl`.

Amidst the chaos, a common set of admin practices seems mandatory so
that each application is looked after in a way that is implementation
agnostic (like you would do with any programming interface). One
does not care *how* these practices are fulfilled, only that they are.
Some can be scripted, some are a matter of documentation, but all are
required for any application to makes it into your production cluster.
One could say that every application can implement its own *methods* for
the same *operations*. Operations are the actions taken by any cloud
native admin, or by a script on behalf of the admin. 

To be clear, this is not yet-another-tool. It's just a matter of
agreeing on what everyone on the admin team should do with respect to
*all* Kubernetes applications while allowing each its own specificity.

The K8SAPP contract simply asks that all team members agree to the
following:

* Maintain manifest of all installed applications in own repo (ex: `k8sapps`)
* Provide a Git repo for each application (allows GitOps, etc.)
* Tag each version using semver versioning (ex: `v0.1.1`)
* Use a consistent prefix (ex: `k8sapp-`)
* Provide a detailed README.md with description, etc.
* Provide a metadata file (ex: `k8sapp.yaml`)
* Provide single resources file named for namespace (ex: `nfd.yaml`)
* Keep a vendored copy of the original application (ex:
  `helm/some-chart-0.8.tgz`, `helm/index.yaml`)
* Understand what the final Kubernetes resources will be before
  installing them
* Pull, vet, and push remote images to private registry
* Document the following procedures (as `##` in README.md):
    * Fetch - acquire from external authoritative source
    * Configure - adapt original to match environment (`values.yaml`)
    * Install - install the application for the first time
    * Upgrade - install an upgraded version of the application
    * Uninstall - remove all traces of the application from the cluster
    * Check - latest version, dependencies, is there an update?
* Add a *## Related* section with related reading
* Keep a manifest of all installed apps in own Git repo (ex: `k8sapps`)

> 💡
> Consider creating a `template-k8sapp` for your organization in your
> Enterprise GitHub so that people can simply clone/copy it and fill it
> in. 

## K8SAPP Embraces Git Instead of Shunning It

One of the most annoying design decisions of Helm is discarding the
world's best source management utility (Git) and replacing it with
tarballs. This is a source of much confusion and extra work when
people realize they still need a way to track all their customizations
and forks to inadequate templates. Had the Helm project had the
foresight to keep all "Charts" as repos this would all go away. We could
just fork a "chart" and make our changes and ensure upstream changes are
made over time. This is a tried and true Git process used by most
applications development for over a decade. For some foolish reason, the
Helm project choose to discard this wisdom.

A K8SAPP undoes what Helm has broken by flattening Helm charts into their
default Kubernetes resource YAML files and putting them into Git repos
for proper management.

## K8SAPP Conventional Metadata

Having consistent metadata in each K8SAPP Git repo allows the consolidation of this data later into a master manifest used to regularly check for version updates and such.
Each Git repo should begin with a specific prefix following the K8SAPP convention
(preferably `k8sapp-`). Each K8SAPP repo should contain a `k8sapp.yaml`
containing the following metadata:

`fullname` unique long name within the scope of a given Git repo collection or service that may contain qualifiers similar to container image references (ex: `rwxrob/jupyterhub`)

`shortname` for referencing easily within local contexts and corresponding to the git repo name after the prefix (`k8sapp-jhub` -> `jhub`)

`title` descriptive title of 70 Unicode characters or less for creating lists of K8SAPPs 

`version` semver of this K8SAPP, *not* it's origin, `git tag` also must be synced

`repo` Git-friendly reference to the required source Git repo (usually HTTP)

`maintainers` optional list of primary contacts with `name`, `email`, `url`, `slack`

*Why no summary or description?*

These should best be placed in the required README.md file within the required Git repo since CommonMark is much more expressive.

*Why no `kubeVersion`?*

The `kubeVersion` in Helm is disastrously broken to the point of being
dangerous. In addition, any indication of a K8SAPP being usable by
a specific version *or greater* is simply a lie since there is no way
to guarantee future compatibility.

Moreover, the Kubernetes version is but one consideration. Some K8SAPPs
may have dependencies on specific versions of other installed
applications.

This complexity is best communicated and managed in the README.md file
itself and should probably not be automated.

## Vendoring (When Necessary)

Vendoring is the process of saving external dependencies with one's own
code. The K8SAPP conventions leverage the vendoring approach to
preserve Helm charts and other YAML resource files that originate
outside of the project. Acquiring these external resources can be
included in a `fetch` action or simply documented in the README.md when
an Internet connection cannot be assumed.

## "Package Management" Will Never Work in Kubernetes

> Kubernetes is not a Linux distro!

The term "package management" (popularized wrongly by the Helm project)
does not relate to Kubernetes and should be avoided. It is causing
immense confusion and chaos in the industry. Use any of the following
terms instead:

* application build utility
* configuration management assistant

Helm is closer to Webpack (a NodeJS build tool) than to APT (the Ubuntu
package manager). Calling Helm a "package manager" is simply a lie.
Let's look at an actual package manager in comparison.

When installing `openssh-server` one need only execute `apt install
openssh-server` and everything works because APT has been designed to
work with Debian/Ubuntu systems and all of those systems --- with regard
to software installation and management --- follow the same standard.
There is no intermediate validation step, no need to comb through the
source code to make sure it works on your computer, no need to add extra
dependencies that are not covered by the APT system itself, no need to
change references from external image dependencies into internal ones,
no need to copy down the package specification file to ensure we have a
copy of it. Everything just works. *That* is what a package manager
provides.

Package managers work because the distros that support them have a
contract to provide what those package managers needs. There is a
concrete agreement between the creators of the distro and the creators
of the package manager. Such a relationship *does not exist* between any
would-be Kubernetes "package manager" nor could there ever be because
every single Kubernetes cluster is completely different.

Consider the process of installing and maintaining a Helm chart. First
you identify the chart, then copy down the tarball, uncompress it, look
through the defaults with `helm template ...` read through the
templates themselves looking for bugs and places to customize, check the
Git repo to see if any of the open issues relates to you, change the
references to external images to internal mirrored copies of the images,
run the resource files through OPA `conftest` to pro-actively ensure
they meet security policies, then attempt to install only to discover
that you must create additional RBAC resources before it will even run,
and then be able to undo all of that customization once you remove or
upgrade your "package". This is *not* package management. This is
software configuration, building, and installation. Helm is *not* a
package manager. It never was. It was born out of the needs for
application creators to make it easier to *build* applications, not
install and manage them. 

Helm had good intentions, but Helm tried to provide an impossible
solution. As much as naive tech-blog writers would have you think
otherwise, Kubernetes is not anything like a Linux distribution. Every
Kubernetes cluster is unique and has its own configurations and
requirements that require a large amount of customization, even between
development and production clusters. This fact makes creating a standard
"package manager" impossible. It would be like creating an APT system
that addressed every single different Linux distro available today. The
fallacy that such applications can be managed in the same simple way is
the source of much wasted time, money, and safety.

One popular attempt at a universal solution to applications management
and installation demonstrates how ludicrous this all is. The Operator
Framework (originally from RedHat) is a disastrous security failure
requiring its OLM to have full cluster admin permissions and placing the
responsibility for cluster security on the administrators who are warned
(despite the open tickets regard this massive security flaw) to "check
anything you install with it to be sure it's safe." Imagine telling that
to APT users. "Before you do that `apt install` first make sure the
package is safe." Keep in mind that Helm 2 with Tiller was shamed out of
existence by the industry for attempting the same thing.

Ironically, even though OLM has the equivalent of (what I call)
enterprise root (root not just to one server, but your whole 
enterprise cluster) it is still unable to provide the simplicity of a
"package manager" we've come to expect when the distro is relatively
locked down. Again, the problem is with the design assumption, not the
technology. Kubernetes is not Linux distro. It's anything but. Please stop
perpetuating this dangerous comparison.

Instead, we should be focused on facilitating duplication of application
code and adding sustainable practices for maintaining that code over
time leveraging the same successful process evolved from software
applications development. Many do not like to hear this, but you cannot
administer a Kubernetes cluster without software development skills,
notably GitOps, CI/CD, shell scripting, Go code review, and YAML/JSON
configuration. Make sure your administrators are keeping these skills up
to date.

## Helm Deployment from Hell, A Reality Check

By now you've probably heard of Helm and used it to deploy software into
your Kubernetes cluster. In fact, in 2022 it is now required learning
for the certification exam. But let's be honest, Helm is horrible.
You've likely learned this yourself. Helm is not what you were sold when
you first heard about it. 

**Helm is not a "package manager"** like `apt` or `yum` or whatever. In
fact, the only thing Helm has in common with a *true* package manager is
the aggregation of lots of stuff that eventually needs to be installed
and a few common top-level usage verbs: `deploy`, `uninstall`, etc.

**Helm charts are not packages.** Charts are just a bunch of YAML and Go
templates organized in no particular way. There's no standard, no
convention, nothing. Every single chart does things slightly differently,
which would be fine except ...

**Helm requires you to read chart files to be sure you get what you
want.** You simply cannot trust a Helm chart all by itself. If you
haven't learned this lesson by now, you definitely will. It is a
fundamental best practice to read just about every line of the Helm
templates and values to actually understand what the chart is going to
do (or not do) to your cluster. Not doing so is simply dangerous. 

**Helm charts are *not* easy to create and frequently contain
problems.** Kubernetes is crazy complicated meaning that those creating
Helm charts inevitably make mistakes simply from the nature of the task
complexity. At a minimum, you must add OPA Gatekeeper policy constraints
that apply to your cluster and are therefore almost never hard-coded
into any chart meaning you either use their escape hatches to add them
or fork the main template and add them directly since they were
omitted, or monkey patch your additions after Helm does its install from
a bash script. This insanity is decidedly different than any "package
manager" where you can safely trust it to do the right things (but not
always). Bottom line: you better at least read your chart code before
installing it and you'll probably have to write more code to do what you
want.

**Helm obfuscates an already complicated Kubernetes resource YAML
syntax.** Just when you got good at "YAML Kubernetes programming" you
realized that Helm chart creators have taken liberties with even the
most consistent stuff, like a Deployment's
`.spec.containers[].resources.limits.cpu` and have buried it under the
whimsical monikers decided by the persons who created the Helm chart.
This is infuriating. Those who created the Helm chart either force you
to alter the chart source code itself (removing the entire value
proposition of putting it into a chart in the first place) or they add
something like `extraConfigs` that you would never guess without having
read a dozen other charts that follow the same non-convention.

"So what is a cloud-native engineer to do?"

**Avoid Helm whenever possible.** This might seem like an unpopular
opinion at first, after all, why is Helm including in the CNCF
certification if it is *not* a standard? But the reality is that most
senior cloud native community members including Kubernetes book authors,
CNCF board members, and senior architects at major Fortune 500
companies advice to avoid Helm wherever possible, even that flattening a
Helm chart is preferred in almost every case.

Why? 

Because there is substantial, objective evidence that creating
deployments that depend on Helm is a disaster waiting to happen. Most
will only do it when forced to do it. It is far simpler to simply write
a Go binary to handle all your installs and uninstalls following a
common, intuitive convention when needed, or just a simple shell script
combined with YAML files. You can still use Go templates and YAML files
to allow people to pass in their configurations and customizations, but
make your main deployment mechanism a single Go binary that can be
easily versioned and managed as the *software* that is it.

**The Helm disaster is not entirely Helm's fault.** The entire
"declarative" model of Kubernetes is fundamentally broken on several
levels with regard to applications installation and management. Things
that should have been written as imperative procedures are written in
complicated, configuration templates with obscure logic instead of YAML
declarations since YAML does not allow imperative logic, which is why
...

**It is Kubernetes' fault Helm is so bad.** The overengineered
complexity and insistence that *everything* be "declarative" in
Kubernetes has forced imperative logic into things like Helm (Go)
templates leaving Helm in the very difficult middle-space between
declarative and imperative. This is why Helm is so ugly. It is a ton of
declarative YAML and imperative templates mixed into an incomprehensible
mess.

**Helm (Go) templates are where the imperative stuff ends up.**
Unfortunately, the most powerful part of Go templates is entirely
omitted: creating your own operators with niladic functions, something
that you *could* do if you didn't use Helm and just wrote your installer
in Go directly (which is the direction NAML seems to be headed). Imagine
if Helm tried to allow template expansion. We would have chart creators
building wildly complicated Go templates, they would even create their
own binary extensions in Go. Writing application installers in Go is
always more powerful and simpler in such cases.

**Why not just write the whole thing as code --- in one language
--- in the first place?** If you are honest, you will admit that you
have at least one script or system that "wraps" Helm to get it to do
what you need. Why not just drop Helm entirely and write your own
installers for your applications in Go or Bash. Go is drop-dead
simple to learn and already has Go templates (if and when you need them).
By packaging your application as a single Go executable you drop tons of
unnecessary abstraction layers and obfuscation that are not only less
sustainable and annoying, but also inherently less secure. The Go
Kubernetes API and package are amazingly simple compared to the
spaghetti code created with Helm (or even YAML resource files
themselves).

**"What about consistency?"** The Helm scenario from Hell I've described
is anything but consistent. You already have best possible consistency
with the Kubernetes API itself and Go package itself. The reason it is
no more work to get your users to understand your single command than it
is to use Helm is because of all the reality that Helm forces on you.
You *are* coding when you install a Helm chart. This is not some 
`apt install yourapp`. You have coded YAML files, and sometimes template
extensions forcing you to "fork" the original chart. You've added
scripts for things that cannot be done in either YAML or templates.

**Helm claims to simplify installation but doesn't.** For all the
reasons mentioned you end up with a complicated, per-chart spaghetti
monster that you have to kill, pick apart, extend, and paste back
together with home-grown bash scripts to get a consistent deployment
working at all. Why not just use *actual* resource files and Go
templates as a best-practice and drop Helm entirely? Just code it in
Bash or Go

## Operator Framework / Operator Lifecycle Manager (OLM) is Dangerous

The Operator Framework is a disastrous, irreparable failure. OLM with
full cluster-admin privileges is just Helm 2 Tiller all over again, but
worse. Not only does OLM require enterprise root (my term for
cluster-admin to your entire cluster) but it also *requires* access to
the Internet to pull operators from their hub to then be run as root.
You can forget about being "air gapped" at all. You would think all the
stuff RedHat spews about Docker and running as root that someone would
have said something before this monstrosity was ever released. No one
should *ever* consider using the Operator Framework, period. In fact, my
confidence in RedHat in the cloud-native space is now at an all-time low
after discovering this. This is just idiotic architecture. They've had a
ticket open for more than a year pointing out this massive security risk
and the responses are downright juvenile:

> ⚠️
> The OLM approach to K8S software installation and management does not
> provide for organizations with "air gapped" policies and architecture.
> It simply does not work without Internet access to `operatorhub.io`.

> ⚠️
> Operator Lifecycle Manager (OLM) runs with cluster-admin privileges.
> ...
> Cluster administrators should take measures to ensure that an Operator
> cannot achieve cluster-scoped privileges.

Here is the ClusterRole requires by OLM:

```yaml
# Source: original/0000_50_olm_01-olm-operator.serviceaccount.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: system:controller:operator-lifecycle-manager
  labels:
    addonmanager.kubernetes.io/mode: Reconcile
rules:
- apiGroups: ["*"]
  resources: ["*"]
  verbs: ["*"]
- nonResourceURLs: ["*"]
  verbs: ["*"]
```

Related:

* Operator Lifecycle Manager  
  <https://olm.operatorframework.io/>
* <https://docs.openshift.com/container-platform/4.9/operators/admin/olm-creating-policy.html>
* <https://github.com/operator-framework/operator-lifecycle-manager/issues/1685>

## Identify and Remediate Rather Than Gatekeep

Recently, I abandoned a mini-project and proposal for a `validate` Helm
plugin that used OPA Gatekeeper to pro-actively check a chart to see
that its configuration passed policies. Then I realized this would need
to be maintained in addition to the OPA Gatekeeper configuration itself.
I realized this is redundant and unnecessary but also not even optimal.
In fact, other more experienced community members questioned my
gate-keeping approach suggest an alternative focus on regularly and
automatically auditing what is currently in the cluster.

It's rather easy to get something past the gates and into your cluster.
It is more important to focus on identifying problems and risks in your
active cluster dynamically. This is what the Falco project aims to do. I
like the emphasis on adaptability which allows dynamic auditing to
adjust parameters much like machine learning security scanning software
in other realms of the industry. Eventually, we'll have bots patrolling
our Kubernetes clusters just like they do everything else. This is really
where I want to focus our attention rather than on dated gate-keeping
approaches.

Related:

* <https://falco.org>

## Examples

* Node Feature Discovery <https://github.com/rwxrob/k8sapp-nfd>
* JupyterHub <https://github.com/rwxrob/k8sapp-jhub>
