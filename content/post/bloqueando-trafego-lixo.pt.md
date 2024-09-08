---
title: "Bloqueando bots, IAs, spam e tráfego lixo do seu site"
slug: "bloqueando-trafego-lixo"
date: 2024-07-29T11:58:25-03:00
draft: false
author: "Thiago Santos"
translationKey: "bloqueando-trafego-lixo"
tags: ["tecnologia","apache","web"]
---



Nem todo tipo de tráfego é legítimo. Muitos [scrappers](https://wikipedia.org/wiki/Web_scraping), robôs, [IAs](https://pt.wikipedia.org/wiki/Intelig%C3%AAncia_artificial) entre outros acessam paginas aleatoriamente. Na melhor das hipóteses, eles só vão consumir recursos do seu [servidor](https://pt.wikipedia.org/wiki/Servidor). Mas há tráfegos advindo de aplicações maliciosas que podem replicar seu conteúdo sem autorização, utilizá-lo para treinar IA, ou até mesmo realizar um um [DOS](https://pt.wikipedia.org/wiki/Ataque_de_nega%C3%A7%C3%A3o_de_servi%C3%A7o). Para evitar que isso ocorra, é necessário bloquear acessos indesejados.

Esse manual em forma de tutorial, não é recomendado para você que quer monetizar sua página, ou simplesmente receber cliques. Esse método bloqueia grande parte do acesso não humano, [Search Engines](https://pt.wikipedia.org/wiki/Motor_de_busca) e de curiosos. Se o seu objetivo é que apenas humanos interessados no seu conteúdo acessem sua página ok, mas se você quer farmar cliques, usar sua página para redirecionar produtos ou tráfego, talvez apenas a parte de bloquear robôs seja util pra você.

Para hospedar esse blog, eu utilizo o [Apache Webserver](https://apache.org/). Nele podemos inserir diretivas de acesso para aceitar ou não o acesso. Há duas maneiras de você configurar isso. Pelo arquivo [.htaccess](https://httpd.apache.org/docs/2.4/howto/htaccess.html), mas a cada requisição ele será carregado e processado, ou diretamente no [httpd.conf](https://httpd.apache.org/docs/2.4/configuring.html), que é o melhor método, pois ele só será carregado no momento que o serviço do Apache subir. Vou utilizar o exemplo do .htaccess, pois a maioria das hospedagens não permitem você acessar o httpd.conf. E de qualquer forma, se você tem estrutura para ter um servidor dedicado ou uma [VPS](https://pt.wikipedia.org/wiki/Servidor_virtual_privado), você terá os conhecimentos para aplicar o que eu fiz com o .htaccess no httpd.conf.

A primeira parte é bloquear [referers](https://pt.wikipedia.org/wiki/HTTP_referer) que são [spam](https://pt.wikipedia.org/wiki/Spam). Há sites que você não vai querer tráfego vindo deles. No meu caso, eu bloqueei quase todas as redes sociais que eu conheço (exceto o Linkedin), pois geralmente, quando alguém está numa rede social e vê um link, no máximo só clica, dá uma olhada e vai embora. E também seu link numa rede social, seu conteúdo vai chegar em várias pessoas que não são o público alvo. Mas se você tem alguma necessidade, só não bloquear.

Também bloqueei tráfego advindo de Searches Engines, como Google, Bing, Yahoo, etc, pois além deles não favorecerem a indexação de conteúdos não comerciais, eles podem usar seu conteúdo para treinamento de IA. E outra, essa página tem meus contatos, se o Google indexar, algum robô que utiliza o Google para achar sites e contatos, vai ficar me flodando de Spam.

Agora vamos bloquear tráfego pelo User-Agent. Basicamente, o User-Agent informa com qual navegador o cliente está usando para conectar na sua página. Se for sites estranhos podemos bloquear.

O arquivo final ficou assim.




    RewriteCond %{HTTP_REFERER} news\.ycombinator\.com [NC,OR]
    RewriteCond %{HTTP_REFERER} ^(.*\.)?google\.com [NC,OR]
    RewriteCond %{HTTP_REFERER} ^(.*\.)?bing\.com [NC,OR]
    RewriteCond %{HTTP_REFERER} ^(.*\.)?twitter\.com [NC,OR]
    RewriteCond %{HTTP_REFERER} ^(.*\.)?facebook\.com [NC,OR]
    RewriteCond %{HTTP_REFERER} ^(.*\.)?fb\.com [NC,OR]
    RewriteCond %{HTTP_REFERER} ^(.*\.)?gov$ [NC,OR]
    RewriteCond %{HTTP_REFERER} ^(.*\.)?gov\.br$ [NC,OR]
    RewriteCond %{HTTP_REFERER} ^(.*\.)?threads\.net [NC,OR]
    RewriteCond %{HTTP_REFERER} ^(.*\.)?instagram\.com [NC,OR]
    RewriteCond %{HTTP_REFERER} ^(.*\.)?bsky\.app [NC,OR]
    RewriteCond %{HTTP_REFERER} ^(.*\.)?x\.com [NC,OR]
    RewriteCond %{HTTP_REFERER} ^(.*\.)?reddit\.com [NC,OR]
    RewriteCond %{HTTP_REFERER} ^(.*\.)?forum\.agoraroad.com [NC,OR]
    RewriteCond %{HTTP_REFERER} ^(.*\.)?kiwifarms\.net [NC]
    #RewriteCond %{HTTP_REFERER} ^(.*\.)?linkedin\.com [NC]
    RewriteRule .* - [F,L]

    # Bloqueia Crawlers,Spiders e IA pelo User Agent

    RewriteCond %{HTTP_USER_AGENT} (spyder) [NC,OR]
    RewriteCond %{HTTP_USER_AGENT} (bot) [NC,OR]
    RewriteCond %{HTTP_USER_AGENT} (crawler) [NC,OR]
    RewriteCond %{HTTP_USER_AGENT} (scrap) [NC,OR]
    RewriteCond %{HTTP_USER_AGENT} (spy) [NC,OR]
    RewriteCond %{HTTP_USER_AGENT} (seo) [NC,OR]
    RewriteCond %{HTTP_USER_AGENT} (wallpaper) [NC,OR]
    RewriteCond %{HTTP_USER_AGENT} (image) [NC,OR]
    RewriteCond %{HTTP_USER_AGENT} (copyright) [NC,OR]
    RewriteCond %{HTTP_USER_AGENT} (check) [NC,OR]
    RewriteCond %{HTTP_USER_AGENT} (spam) [NC,OR]
    RewriteCond %{HTTP_USER_AGENT} (spanner) [NC,OR]
    RewriteCond %{HTTP_USER_AGENT} (scan) [NC]
    RewriteRule .* - [F,L]






Esse arquivo bloqueia até redes sociais, mas você pode alterar se desejar.

