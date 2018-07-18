---
title: Gist 002
tags:
  - G-02
---

The Globex Corporation is a company run by Hank Scorpio. Disguised as a friendly and innocent high-tech company,
Globex is actually a front for an evil organization that Hank Scorpio uses to take over the East Coast of America.

{% highlight html %}
  <div class="card mb-3">
      <img class="card-img-top" src = "http://via.placeholder.com/900x250/20c997/ffffff?text=screenshot"/>
      <div class="card-body bg-light">
          <div class="card-text">Screenshot of my work.</div>
      </div>
  </div>
{% endhighlight %}

{% highlight ruby %}

  def project do
    [
      app: :linklab_app,
      version: @version,
      elixir: @elixir_version,
      elixirc_paths: elixirc_paths(Mix.env()),
      compilers: [:phoenix] ++ Mix.compilers(),
      start_permanent: Mix.env() == :prod,
      aliases: aliases(),
      deps: deps(),
      dialyzer: dialyzer(),
      test_coverage: [tool: ExCoveralls],
      preferred_cli_env: [coveralls: :test, "coveralls.detail": :test, "coveralls.html": :test],
      docs: docs()
    ]
  end

{% endhighlight %}