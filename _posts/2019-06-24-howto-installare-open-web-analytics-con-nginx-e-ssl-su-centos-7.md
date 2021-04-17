---
title: '#howto - Installare Open Web Analytics con Nginx e SSL su Centos 7'
published: 2019-06-24
layout: post
author: Mirko B.
author_github: mirkobrombin
tags:
  - nginx  - mysql  - centos  - letsencrypt  - php  - github  - bash
---
<p>OWA (Open Web Analytics) è una piattaforma di analitca performante e completa, oltre che Open source.</p><blockquote><p>Abbiamo già parlato di Open Web Analytics e altre piattaforme di analitica, in <a href="https://linuxhub.it/article/le-migliori-piattaforme-web-analytics-open-source#title2">questo articolo</a>.</p></blockquote><p>In questa guida vediamo come installarla su Centos 7, tramite web server Nginx.</p><h2>Requisiti</h2><p>Prima di procedere è opportuno consultare i requisiti annunciati dal team di sviluppo:</p><ul>	<li>Un Web Server (in questa guida Nginx)</li>	<li>MySQL 4.1+</li>	<li>PHP 5.2+</li></ul><h2 id="title1">Configurazione dominio</h2><p>Il dominio è sostanzialmente&nbsp;"la base del link" da cui vogliamo raggiungere la nostra installazione. Un esempio è proprio questo sito web, raggiungibile appunto da: linuxhub.it.</p><p>Per una miglior&nbsp;fruibilità dei contenuti, la guida su come <strong>puntare un dominio ad un IP</strong>&nbsp;è disponibile <a href="https://linuxhub.it/article/howto-puntare-un-dominio-ad-un-ip">a questo link</a>.</p><h2>Installazione di php-fpm</h2><p>Iniziamo dall'installazione di php ed i moduli necessari come da requisiti:</p><pre><code class="language-bash">sudo yum install php php-fpm</code></pre><p>proseguiamo poi con l'installazione dei&nbsp;moduli:</p><pre><code class="language-bash">sudo yum install php-gd php-ldap php-odbc php-pear php-xml php-xmlrpc php-mbstring php-snmp php-soap curl curl-devel php-json</code></pre><p>in questo modo la nostra configurazione sarà compatibile con una installazione base della piattaforma.</p><p>Infine avviamo il processo ed abilitiamo l'auto esecuzione all'avvio del server:</p><pre><code class="language-bash">sudo systemctl start php-fpm sudo systemctl enable php-fpm</code></pre><h2 id="title6">Installazione di Nginx</h2><p>Per prima cosa aggiungiamo la repository ufficiale di Nginx, in modo da ottenere una versione aggiornata. Creiamo quindi il file <strong>nginx.repo</strong> col nostro editor preferito:</p><pre><code class="language-bash">sudo nano /etc/yum.repos.d/nginx.repo</code></pre><p>con il seguente contenuto:</p><pre><code class="language-bash">[nginx] name=nginx repo baseurl=http://nginx.org/packages/centos/$releasever/$basearch/ gpgcheck=0 enabled=1</code></pre><p>proseguiamo infine con l'installazione:</p><pre><code class="language-bash">sudo yum install nginx</code></pre><p>e l'abilitazione ed avvio del servizio via systemctl:</p><pre><code class="language-bash">sudo systemctl enable nginx sudo systemctl start nginx</code></pre><h2>Installazione di MariaDB</h2><p>In questa guida usiamo MariaDB come Database Server, procediamo quindi alla sua installazione via yum:</p><pre><code>sudo yum install mariadb mariadb-server</code></pre><p>come per i precedenti, abilitiamo ed avviamo il servizio:</p><pre><code>sudo systemctl enable mariadbsudo systemctl start mariadb</code></pre><p>Proseguiamo poi la configurazione guidata:</p><pre><code>sudo mysql_secure_installation</code></pre><p>e seguiamo le istruzioni a schermo, ricordando di annotare la&nbsp;<strong>password</strong>&nbsp;una volta fornita poichè ci servirà nella fase di installazione.</p><blockquote><p>Personalmente consiglio le seguenti scelte nella fase di configurazione di mysql:</p><ul>	<li>Set root password? Si</li>	<li>Remove&nbsp;anonymous users? Si</li>	<li>Disallow root login remotely? Si</li></ul></blockquote><h3>Creazione Database e user</h3><p>Ora che tutto è installato e configurato, procediamo con la creazione del database vero e proprio oltre che dell'utente con cui accedervi. Useremo questi dati nella fase di installazione della piattaforma.</p><p>Facciamo accesso alla <strong>console</strong> <strong>mysql</strong> digitando&nbsp;quindi:</p><pre><code class="language-bash">mysql -u root -p</code></pre><p>ci verrà chiesta la password inserita in fase di configurazione di mysql, inseriamola e creiamo il nuovo database:</p><pre><code class="language-sql">CREATE DATABASE il_mio_owa;</code></pre><p>dove&nbsp;<strong>il_mio_dominio</strong>&nbsp;sarà il nome del database. Creiamo ora l'utente, digitiamo la query:</p><pre><code class="language-sql">CREATE USER il_mio_user@localhost IDENTIFIED BY 'la_mia_password';</code></pre><p>dove:</p><ul>	<li>il_mio_user - è il nome utente con cui faremo accesso al database</li>	<li>la_mia_password - è la password del nostro utente</li></ul><p>diamo ora i permessi di accesso al nostro utente con la seguente query:</p><pre><code class="language-sql">GRANT ALL PRIVILEGES ON il_mio_owa.* TO il_mio_user@localhost;</code></pre><pre><code class="language-sql">FLUSH PRIVILEGES;</code></pre><p>e chiudiamo la console:</p><pre><code class="language-sql">exit</code></pre><h2>Configurazione di php-fpm</h2><p>Procediamo con la configurazione di php-fpm, per renderlo "compatibile" col nostro server nginx. Modifichiamo il file in locazione <strong>/etc/php-fpm.d/www.conf</strong>:</p><pre><code>sudo nano /etc/php-fpm.d/www.conf</code></pre><p>modificando i seguenti campi come da esempio:</p><pre><code>...user = nginxgroup = nginx...listen = /run/php-fpm/www.sock...</code></pre><p>Infine creiamo la cartella per le sessioni di PHP ed impostiamo i permessi all'utente nginx:</p><pre><code>sudo mkdir -p /var/lib/php/sessionsudo chown nginx:nginx -R /var/lib/php/session/</code></pre><p>riavviamo il servizio:</p><pre><code>sudo systemctl restart php-fpm</code></pre><h2>Certificato SSL</h2><p>Procediamo con la creazione di un certificato SSL, in modo da sfruttare il protocollo https sicuro.</p><blockquote><p>Nel caso in cui utilizzi sistemi come Cloudflare, ti basterà generare il certificato direttamente dalla Dashboard per poi importarlo sul server.</p></blockquote><p>Per una migliore gestione dei contenuti, questa parte di guida è stata spostata <a href="https://linuxhub.it/article/howto-ottenere-e-rinnovare-un-certificato-ssl-con-lets-encrypt">qui</a>.</p><h2>Configurazione di Nginx</h2><p>Andiamo ora a creare un file di configurazione Nginx per il sito che ospiterà la piattaforma OWA:</p><pre><code class="language-bash">sudo nano /etc/nginx/conf.d/il_mio_sito.ex.conf</code></pre><p>dove al suo interno poniamo il seguente contenuto:</p><pre><code class="language-nginx">server {    listen 443 ssl http2;    listen [::]:443 ssl http2;    server_name  il_mio_sito.ex www.il_mio_sito.ex;    ssl_certificate     /etc/letsencrypt/il_mio_sito.ex.pem;    ssl_certificate_key /etc/letsencrypt/il_mio_sito.ex.key;        client_max_body_size 100M;    root   public/il_mio_sito.ex;    index  index.php index.html index.htm;        location / {        try_files $uri $uri/ /index.php?q=$uri&amp;$args;    }    location ~ \.php$ {        try_files $uri =404;        fastcgi_pass unix:/run/php-fpm/www.sock;        fastcgi_index index.php;        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;        include fastcgi_params;    }}</code></pre><p>dove:</p><ul>	<li><strong>server_name </strong>è il nome a dominio che abbiamo configurato e dedicato alla nostra installazione</li>	<li><strong>ssl_certificate</strong> è la locazione del certificato SSL precedentemente generato</li>	<li><strong>ssl_certificate_key </strong>è la locazione del chiave del certificato SSL</li>	<li><strong>root </strong>è la locazione dove andremo ad aggiungere i file di installazione</li></ul><p>Creiamo ora la locazione per i file di installazione:</p><pre><code>sudo mkdir -p /usr/share/nginx/html/il_mio_sito.ex</code></pre><p>ed impostiamo i permessi all'utente nginx:</p><pre><code class="language-bash">sudo chown -R nginx:nginx /usr/share/nginx/html/il_mio_sito.ex</code></pre><p>infine riavviamo Nginx:</p><pre><code class="language-bash">sudo systemctl restart nginx</code></pre><h2>Installazione di Open Web Analytics</h2><p>Portiamoci alla locazione precedentemente creata, nel nostro esempio /usr/share/nginx/html/il_mio_sito.ex:</p><pre><code class="language-bash">cd /usr/share/nginx/html/il_mio_sito.ex</code></pre><p>Installiamo <strong>wget</strong> per il download delle risorse:</p><pre><code class="language-bash">sudo yum install wget</code></pre><p>Scarichiamo l'ultima versione di Open Web Analytics (la <strong>1.6.2</strong> nel momento in cui scrivo) dalla <a href="https://github.com/padams/Open-Web-Analytics/tags">repository</a> ufficiale:</p><pre><code class="language-bash">wget https://github.com/padams/Open-Web-Analytics/archive/1.6.2.zip</code></pre><p>ora scompattiamo l'archivio scaricato e spostiamo il suo contenuto nella cartella in cui siamo:</p><pre><code class="language-bash">unzip *.zipmv Open-Web-Analytics*/* ./</code></pre><p>e impostiamo i permessi corretti:</p><pre><code class="language-bash">sudo chown -R nginx: nginx ./sudo chmod -R 777 owa-data</code></pre><p>Ora visitiamo il dominio che abbiamo configurato e procediamo con l'installazione guidata.</p><p>Consiglio di proseguire la lettura con la <a href="https://github.com/padams/Open-Web-Analytics/wiki/Configuration">Wiki ufficiale</a> di Open Web Analytics, per una configurazione ottimale.</p><p>&nbsp;</p><p><em>Good&nbsp;<strong>*nix</strong>?</em><br /><em>&nbsp;- Mirko</em></p>