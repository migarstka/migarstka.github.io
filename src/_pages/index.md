---
layout: index
permalink: /
---

### About me

I am a DPhil (PhD) student at the [Department of Engineering Science](http://www.eng.ox.ac.uk/), [University of Oxford](http://www.ox.ac.uk/) under the supervision of [Prof. Paul Goulart](http://users.ox.ac.uk/~engs1373/) and [Prof. Mark Cannon](http://www.eng.ox.ac.uk/control/people/dr-mark-cannon) in the [Control Group](http://www.eng.ox.ac.uk/control). I am associated with [Trinity College](https://www.trinity.ox.ac.uk/) and supported by the [Clarendon Scholarship](http://www.ox.ac.uk/clarendon).

### Research interests

I am interested in model predictive control and optimization. My current project investigates ADMM algorithms to solve convex conic programs. I am especially focused on improving solver performance using chordal decomposition techniques and acceleration methods. I am also interested in applications of (convex) optimisation such as Portfolio Optimisation, Machine Learning, Optimal Control and Graph Theory.

Following the worldwide COVID-19 outbreak in the spring of 2020, I joined the [OxVent](https://oxvent.org/) team to work on the control and software for a new rapidly deployable low-cost ventilator for COVID-19 patients. Our work has been recognized with an [E&T Innovation Award](https://www.theiet.org/media/press-releases/press-releases-2020/20-november-2020-covid-19-technology-wins-et-innovation-award/) in the category *Small Idea, Big Impact: Global Challenge*.

### Software
<b>[COSMO.jl](https://github.com/oxfordcontrol/COSMO.jl)</b>   <a href="https://github.com/oxfordcontrol/COSMO.jl/actions"><img src="https://github.com/oxfordcontrol/COSMO.jl/workflows/ci/badge.svg?branch=master"></a>: An ADMM-based solver for convex conic problems. Supports any combination of quadratic cost function and constraints with standard cones. Supports automatic chordal decomposition and clique merging of structured SDPs. A number of example problems can be found [here](https://oxfordcontrol.github.io/COSMO.jl/dev/). A [Python interface](https://github.com/oxfordcontrol/cosmo-python) is also available.

### Notes
<ul>
  {% for post in site.posts %}
    <li>
      <a href="{{ post.url }}">
        {{ post.title }}
      </a>
      - <time datetime="{{ post.date | date: "%Y-%m-%d" }}">{{ post.date | date_to_long_string }}</time>
    </li>
  {% endfor %}
</ul>


### Background

* **since 2017**: DPhil Candidate in Engineering, [University of Oxford](http://www.ox.ac.uk/)
* **2014-2017**: M.Sc. in Theoretical Mechanical Engineering, [Hamburg University of Technology](https://www.tuhh.de/alt/tuhh/startpage.html)
* **2016**: Visiting Student Researcher, [Model Predictive Control Lab](http://www.mpc.berkeley.edu/), [University of California Berkeley](http://www.berkeley.edu/)
  * Master Thesis on Learning Model Predictive Control for Autonomous Driving (with Ugo Rosolia and Francesco Borrelli) [[Video]](https://www.youtube.com/watch?v=Z1q9PjCnIdY)
* **2014**: Visiting Student, Department of Mechanical Engineering, [University of California Berkeley](http://www.berkeley.edu/)
* **2011-2014**: B.Sc. in General Engineering Sciences, [Hamburg University of Technology](https://www.tuhh.de/alt/tuhh/startpage.html)


### Talks

* I (virtually) presented extended results on clique-graph based merging in the Oxford Control Group Journal Club (Dec 4 2020) [[Slides]](/assets/downloads/presentations/2020/garstka_journalclub_clique_merging_presentation.pdf) 

* I (virtually) presented our [paper](https://arxiv.org/abs/1911.05615) on chordal decomposition and clique merging for first order methods at the <b>[21st IFAC World Congress](https://www.ifac2020.org/)</b> 2020 (July 12-17) [[Slides]](/assets/downloads/presentations/2020/garstka_ifac2020_presentation.pdf) [[Video]](https://vimeo.com/439962112)

* I am excited to give a talk at the <b>[fourth JuMP-dev workshop](https://jump.dev/meetings/louvain2020/)</b> 2020 in Louvain-la-Neuve, Belgium (June 15-17) (postponed to 2021)

* I spoke at the <b>[International Conference on Continuous Optimization (ICCOPT)](https://iccopt2019.berlin)</b> 2019 in Berlin, Germany (August 5-8) [[Slides]](/assets/downloads/presentations/2019/garstka_iccopt2019_presentation.pdf)

* I gave a presentation at the <b>[European Control Conference (ECC)](https://ecc19.eu/)</b> 2019 in Naples, Italy (June 25-28).

* I presented COSMO at the <b>[Oxford University SIAM-IMA Student Chapter Conference](https://people.maths.ox.ac.uk/siamsc/con_home.html)</b> 2019 in Oxford, UK (May 1st).

* I gave a talk at the <b>[JuMP-dev workshop](http://www.juliaopt.org/meetings/santiago2019/)</b> 2019 in Santiago, Chile (March 12-14) [[Video]](https://www.youtube.com/watch?v=H4Q0ZXDqB70)

### Publications
{% bibliography %}


### Teaching
<table>
<tr>
<td>Hilary 2019</td>
<td>Demonstrator - 3<sup>rd</sup> year Laboratory <i>Control Systems</i></td>
</tr>
<tr>
<td>Hilary 2019</td>
<td>Demonstrator - 2<sup>nd</sup> year Laboratory <i>Instrumentation and Control</i></td>
</tr>
<tr>
<td>Michelmas 2018</td>
<td>Tutor - 3<sup>rd</sup> year Tutorials <i>Control Systems</i></td>
</tr>
<tr>
<td>Hilary 2018</td>
<td>Demonstrator - 3<sup>rd</sup> year Laboratory <i>Control Systems</i></td>
</tr>
<tr>
<td>Hilary 2018</td>
<td>Demonstrator - 2<sup>nd</sup> year Laboratory <i>Instrumentation and Control</i></td>
</tr>
<table>
<!--
<dl class="dl-horizontal">
  <dt>Hilary Term 2019</dt><dd>Demonstrator - 2<sup>nd</sup> year Laboratory <i>Instrumentation and Control</i></dd>
    <dt>Hilary Term 2019</dt><dd>Demonstrator - 3<sup>rd</sup> year Laboratory <i>Control Systems</i></dd>
  <dt>Michelmas Term 2018</dt><dd>Tutor - 3<sup>3d</sup> year Tutorials <i>Control Systems</i></dd>
  <dt>Hilary Term 2018</dt><dd>Demonstrator - 2<sup>nd</sup> year Laboratory <i>Instrumentation and Control</i></dd>
    <dt>Hilary Term 2018</dt><dd>Demonstrator - 3<sup>rd</sup> year Laboratory <i>Control Systems</i></dd>
</dl> -->
