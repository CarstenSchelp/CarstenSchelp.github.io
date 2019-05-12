---
layout: default
---
<header>
<script type="text/javascript"
  src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.1/MathJax.js?config=TeX-AMS-MML_HTMLorMML">
</script>
<script>
        MathJax.Hub.Config({
            config: ["MMLorHTML.js"],
            extensions: ["tex2jax.js","TeX/AMSmath.js","TeX/AMSsymbols.js"],
            jax: ["input/TeX"],
            tex2jax: {
                inlineMath: [ ['$','$'], ["\\(","\\)"] ],
                displayMath: [ ['$$','$$'], ["\\[","\\]"] ],
                processEscapes: false
            },
            TeX: {
                TagSide: "right",
                TagIndent: ".8em",
                MultLineWidth: "85%",
                equationNumbers: {
                   autoNumber: "all",
                },
                unicode: {
                   fonts: "STIXGeneral,'Arial Unicode MS'"
                }
            },
            showProcessingMessages: true
        });
</script>
</header>
<div style="background-image:url({{ './img/unsupbanner.jpg' | absolute_url }});font-size:150%;color:white;padding:10px;font-family: Helvetica, Arial, sans-serif;">
<div style="font-size:larger; font-weight:bold;">UNSUPERVISED INTUITION</div>
<div style="font-size:medium;text-align:right">
<span>by Carsten Schelp</span>
<span>
<a href="{{ site.linkedin }}"/>
<img alt="LinkedIn" src="{{ './img/linkedinicon.png' | absolute_url }}" style="border:0px;margin:0px;padding:0px;position:relative;left:22px;z-index:+1" width="22" height="22" />
<img alt="Carsten Schelp" src="{{'./img/Carsten_Schelp.jpg'}}" style="border:0px;margin:0px;padding:0px;border-radius:50%" width="80" height="80"/>
</a>
</span>
</div>
</div>
{{ content }}
{% include analytics.html %}



