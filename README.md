# docker-MediaCenter
<hr id="bkmrk-">
<h4 id="bkmrk-pr%C3%A9-requis-%3A">Pré-requis :</h4>
<p id="bkmrk-docker-d%27installer-s">docker d'installer sur votre machine.</p>
<hr id="bkmrk--0">
<h4 id="bkmrk-le-docker-compose-%3A">Le docker-compose :</h4>
<p id="bkmrk-ce-yaml-suivant-cont">Ce yaml suivant contient un ensemble de container pour votre solution de mediacenter :</p>
<ul id="bkmrk-deluge-vpn---%3E-clien">
<li style="list-style-type: none;">
<ul>
<li>Deluge VPN --&gt; client VPN</li>
<li>Jackett --&gt; un seul tracker pour tout vos tracker preferer</li>
<li>Netdata --&gt; Monitorer votre mediacenter</li>
<li>Plex --&gt; Solution de MediaCenter</li>
<li>Portainer --&gt; Container manadgement via webui</li>
<li>Radarr --&gt; automatiser la récuperation de vos films</li>
<li>Sonarr --&gt; automatiser la récuperation de vos series</li>
<li>Watchtower ---&gt; auto update vos containers</li>
</ul>
</li>
</ul>
<p id="bkmrk-avant-de-lancer-le-y">avant de lancer le yaml, veuiller remplacer les variables par vos chemins pour stocker vos datas ainsi que la variable ${IP_ADDRESS}: par rien dutout en laissant que les ports ! </p>
<p id="bkmrk-enregister-le-yaml-s">enregister le yaml suivant avec le nom suivant : docker-compose.yaml></p>
<p id="bkmrk-lancer-votre-stack-%3A">lancer votre stack :</p>
<pre id="bkmrk-docker-compose-up--d"><code class="language-shell">docker-compose up -d</code></pre>
