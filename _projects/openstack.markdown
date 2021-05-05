---
layout: page
title: Review Dynamics in OpenStack
description: A case study on OpenStack to understand how visible information (e.g., prior votes and comments) affects code review decision and the quality of approved patches.
img: /assets/img/openstack-logo.jpeg
importance: 1
category: school
github: https://github.com/david-siqi-liu/review-dynamics-openstack
---

<div class="row">
    <div class="col-sm mt-3 mt-md-0 text-center">
        <img class="img-fluid rounded z-depth-1" src="{{ '/assets/img/projects/openstack/cover.jpeg' | relative_url }}" alt="cover" title="cover">
    </div>
</div>
<div class="caption">
Photo by Christina Morillo from Pexels
</div>

***

## Overview

Code review is an integral part of modern software engineering practice.

In open-source software, code review is done in a collaborative setting where <mark>all review activities are visible to the public, including future reviewers</mark>. This makes us wonder if ...

- Future reviewers tend to agree with prior reviewers?
- Junior reviewers tend to agree with experienced reviewers?
- Developers who collaborate more often tend to agree with each other?

As such, as part of my software analytics course at uWaterloo taught by Prof. Shane McIntosh, I conducted a case study on <a href="https://www.openstack.org/">OpenStack</a> to understand <mark>how visible information may influence the evaluation decision of a reviewer and ultimately, the quality of the approved patches</mark>.

My results show that:

1. Reviewers are more likely to provide a positive vote when there are existing positive votes 😱
2. Patch quality is not as affected by such review dynamics 🥰
3. When a patch is smaller and less complex, it is more likely to receive a positive vote and is less likely to cause defects in the future. So, keep your commits small and push often!

Replication package is available <a href="https://github.com/david-siqi-liu/review-dynamics-openstack">here</a>.

***

## OpenStack

OpenStack is a free, open standard cloud operating system. Since its inception in 2010 as a joint project of Rackspace Hosting and NASA, OpenStack has grown substantially and is now being actively maintained by more than 500 companies and thousands of community volunteers. OpenStack is broken up into service areas and projects. As of now, there are 29 individual projects within OpenStack. Even though each project is managed and developed independently, they share common development infrastructures.

<div class="row">
    <div class="col-sm mt-3 mt-md-0 text-center">
        <img class="img-fluid rounded" src="{{ '/assets/img/projects/openstack/code-review-process.png' | relative_url }}" alt="code-review-process" title="code-review-process" width="500"/>
    </div>
</div>
<div class="caption">
OpenStack/nova code review process flowchart
</div>

The development workflow at OpenStack is centred around Gerrit, which uses the idea of <u>patches rather than pull requests</u>. A quick rundown on how the code review process works:

1. To propose changes to a project, an author starts by cloning the _master_ branch of the project repository
2. After changes have been made locally, the author can propose the changes to Gerrit using the _git-review_ tool, creating a patch
3. The proposed changes are then picked up by the Continuous Integration (CI) tool (either Zuul or Jenkins), which runs _check tests_ on the changes, while human reviewers review and vote on the patch. Depending on the check test results and the feedback from the reviewers, the patch author may need to amend the changes and propose additional patchsets
4. Once the patch has been approved by both the CI tool and the reviewers, the CI tool runs _gate tests_ to detect any merge conflict before finally merging it into the _master_ branch

For this case study, I selected four core projects within OpenStack:

<div class="row">
    <div class="col-sm mt-3 mt-md-0 text-center">
        <img class="img-fluid rounded" src="{{ '/assets/img/projects/openstack/studied-projects.png' | relative_url }}" alt="studied-projects" title="studied-projects"/>
    </div>
</div>
<div class="caption">
</div>

Note that I only used patches submitted from November 2011 to July 2019. This is to eliminate uncertainties from early adoption and to allow enough time for the patches to be reviewed and tested for defects after they have been merged.

Some observations:
- Nearly all of the patches have multiple reviewers, and the average number of reviewers is quite consistent across the projects
- Around 90% of the reviews end up with a positive vote, which is expected since patch authors often make multiple amendments with the help of reviewers to get their proposed changes accepted
- More than half of the patches end up causing defects in the future

***

## Evaluation Decision

The first analysis I did was understanding how visible information, such as prior votes and comments, affects the evaluation decision of a reviewer.

I used <a href="https://pydriller.readthedocs.io/en/latest/">PyDriller</a> to mine patches and extract their characteristics from Git repositories. For each patch, I collected its review dynamics data using the <a href="https://docs.opendev.org/opendev/system-config/latest/gerrit.html">Gerrit REST API</a>. Lastly, I computed reviewer characteristics and assign them to each review.

<div class="row">
    <div class="col-sm mt-3 mt-md-0 text-center">
        <img class="img-fluid rounded" src="{{ '/assets/img/projects/openstack/data-extraction.png' | relative_url }}" alt="data-extraction" title="data-extraction"/>
    </div>
</div>
<div class="caption">
Data extraction pipeline
</div>

I assigned the variables into three data dimensions - `patch`, `dynamic`, and `reviewer`. A detailed list of variables and their descriptions can be found <a href="https://github.com/david-siqi-liu/review-dynamics-openstack/blob/main/report/supplementary_tables.pdf">here</a>.

<div class="row">
    <div class="col-sm mt-3 mt-md-0 text-center">
        <img class="img-fluid rounded" src="{{ '/assets/img/projects/openstack/rq1-variables.png' | relative_url }}" alt="rq1-variables" title="rq1-variables"/>
    </div>
</div>
<div class="caption">
Variables for analyzing evaluation decision
</div>

I conducted correlation analysis to remove highly correted variables. The metric I used is **Spearman’s rank correlation coefficient** ($$\rho$$). For each pair of variables with an absolute correlation coefficient greater than 0.7, I removed one of the variables from our study. For consistency, I used the <a href="https://rdrr.io/github/software-analytics/Rnalytica/man/AutoSpearman.html">AutoSpearman</a> function in R.

<div class="row">
    <div class="col-sm mt-3 mt-md-0 text-center">
        <img class="img-fluid rounded" src="{{ '/assets/img/projects/openstack/rq1-spearman.png' | relative_url }}" alt="rq1-spearman" title="rq1-spearman"/>
    </div>
</div>
<div class="caption">
AutoSpearman correlation graph
</div>

I used **Linear Mixed-Effects Model** as the base model type. Linear mixed-effect model is an extension to simple linear model to account for both fixed and random effects. This type of model is used when there is a hierarchical structure to the data, in which case the variability in the outcome may come from either within the group or between the groups. In our case, the random-effects/groups are the reviewers. For example, some reviewers may be more lenient and tend to give more positive votes than other reviewers.

Since my goal was to predict whether a vote is positive or not, I used `family='binomial'` in my model configuration. To add the random effect, I included `(1|ReviewerID)` in the model formulas.
I fitted five models to explore the explanatory power of each of the three data dimensions. The full model includes variables from all three data dimensions and the random-effect variable. Three models, each excluding one of the data dimensions, were also fitted. Lastly, I fitted a null model which only contains the random-effect variable.

<div class="row">
    <div class="col-sm mt-3 mt-md-0 text-center">
        <img class="img-fluid rounded" src="{{ '/assets/img/projects/openstack/rq1-formulas.png' | relative_url }}" alt="rq1-formulas" title="rq1-formulas"/>
    </div>
</div>
<div class="caption">
Linear mixed-effects model formulas
</div>

For evaluation, I used **Area Under the ROC Curve (AUC)** scores. Both the _Ex-Reviewer_ model and the _Full_ model were able to achieve an AUC score that was 14% higher than the _Null_ model. This implies that even if I excluded all of the `reviewer` variables, the model performance wouldn’t have deteriorated as much.

<div class="row">
    <div class="col-sm mt-3 mt-md-0 text-center">
        <img class="img-fluid rounded" src="{{ '/assets/img/projects/openstack/rq1-auc.png' | relative_url }}" alt="rq1-auc" title="rq1-auc"/>
    </div>
</div>
<div class="caption">
</div>

I conducted **Likelihood-Ratio (LR)** tests to evaluate the explanatory power of each of the data dimensions. While all three data dimensions were statistically significant, `dynamic` variables offered significantly more explanatory ability than variables in the other two data dimensions.

<div class="row">
    <div class="col-sm mt-3 mt-md-0 text-center">
        <img class="img-fluid rounded" src="{{ '/assets/img/projects/openstack/rq1-lr.png' | relative_url }}" alt="rq1-lr" title="rq1-lr"/>
    </div>
</div>
<div class="caption">
</div>

Based on results from the two tests above, I selected the _Ex-Reviewer_ model to be the final model. I calculated the **Wald statistics** for each variable in our final model. The larger the Wald statistics a variable possesses, the stronger its relationship with the target variable.

<div class="row">
    <div class="col-sm mt-3 mt-md-0 text-center">
        <img class="img-fluid rounded" src="{{ '/assets/img/projects/openstack/rq1-wald.png' | relative_url }}" alt="rq1-wald" title="rq1-wald"/>
    </div>
</div>
<div class="caption">
</div>

To examine the generalizability of the final model, I tested it against the validation project. Since I used a mixed-effect model with reviewers being the groups, the model had not seen the reviewers in the validation project and only the fixed-effect variables could have been used for prediction. I achieved an AUC score of 0.73 on the validation project, as opposed to the 0.82 AUC score we saw on the training set. <mark>This implies that there is enough variability among the reviewers that we are unable to form accurate predictions by using the fixed-effect variables alone.</mark>

<div class="row">
    <div class="col-sm mt-3 mt-md-0 text-center">
        <img class="img-fluid rounded" src="{{ '/assets/img/projects/openstack/rq1-roc.png' | relative_url }}" alt="rq1-roc" title="rq1-roc"/>
    </div>
</div>
<div class="caption">
Receiver operating characteristic (ROC) curves
</div>

<mark>Review dynamics have a stronger relationship with the evaluation decision than patch characteristics do.</mark> Even though the `dynamics` data dimension only has 5 variables, it captures 68% of the total LR. Moreover, `% Prior Votes Positive` is the most statistically significant variable overall. Its positive sign indicates that it is more likely for a reviewer to provide a positive vote when there are more positive votes among prior votes.

<mark>Reviewers are more likely to provide a positive vote when the patch changes are smaller and simpler.</mark> This is evident in the signs in `# Lines Added` (-), `# Lines Deleted` (+), and `Entropy` (-).

In addition, the patch author’s experience level and core developer status also have a positive relationship with the likelihood of receiving a positive vote.

***

## Software Quality

I conducted another set of analyses, focusing on the approved patches and their tendency to induce fixes in the future. A patch is fix-inducing if there is a future patch that is bug fixing and modifies the same line (based on the <a href="https://dl.acm.org/doi/10.1145/1082983.1083147">SZZ algorithm</a>).

<mark>Review dynamics do not have as strong of a relationship with software quality as patch characteristics do.</mark> With 62% of the total LR captured, `patch` data dimension has the most explanatory power concerning software quality. `Average Prior Commits Age` is the most significant variable overall. Its positive sign indicates that a patch is more prone to defects when it modifies lines that haven’t been modified for a long time. Other variables, such as `# Lines Added` and `Entropy`, suggest that a patch is more prone to defects when its modifications are larger and more complex. Lastly, I found that whether the author is experienced or not is not statistically significantly associated with the patch quality.

***

## Conclusion

Code review in open source software is done in a collaborative and transparent setting. However, based on the case study, I found that such visible information may have an impact on the evaluation decisions of the reviewers. I showed empirical evidence that review dynamics, particularly the proportion of prior votes that are positive, are highly correlated with the evaluation decision. On the other hand, these review dynamics have a minor impact on the software quality. The results on patch characteristics suggest that developers should propose less complex patches to improve software quality and the likelihood of receiving positive votes. Furthermore, I showed that due to the variability within the reviewers, my model for evaluation decision may not be generalizable to other projects, even within the same ecosystem. Nevertheless, I believe that the findings would be still of value to practitioners looking to improve their code review processes.
