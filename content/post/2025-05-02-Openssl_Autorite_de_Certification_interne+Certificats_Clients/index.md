+++
title = 'Openssl - Création d'une Autorité de Certification interne et de Certificats Clients'
date = 2025-05-02 18:00:00
categories = ['html']
+++
<html xmlns="http://www.w3.org/1999/xhtml" lang="en-US">
<!--This file was converted to xhtml by LibreOffice - see https://cgit.freedesktop.org/libreoffice/core/tree/filter/source/xslt for the code.-->

<head profile="http://dublincore.org/documents/dcmi-terms/">
<meta http-equiv="Content-Type" content="application/xhtml+xml; charset=utf-8"/>
<title xml:lang="en-US">yannick</title>
<meta name="DCTERMS.title" content="yannick" xml:lang="en-US"/>

<meta name="DCTERMS.language" content="en-US" scheme="DCTERMS.RFC4646"/>
<meta name="DCTERMS.source" content="http://xml.openoffice.org/odf2xhtml"/>

<meta name="DCTERMS.issued" content="2025-04-26T15:09:28.031296198" scheme="DCTERMS.W3CDTF"/>

<meta name="DCTERMS.modified" content="2025-05-02T21:12:30.024267973" scheme="DCTERMS.W3CDTF"/>


<meta name="xsl:vendor" content="libxslt"/>
<link rel="schema.DC" href="http://purl.org/dc/elements/1.1/" hreflang="en"/>
<link rel="schema.DCTERMS" href="http://purl.org/dc/terms/" hreflang="en"/>
<link rel="schema.DCTYPE" href="http://purl.org/dc/dcmitype/" hreflang="en"/>
<link rel="schema.DCAM" href="http://purl.org/dc/dcam/" hreflang="en"/>

<style>
    table { border-collapse:collapse; border-spacing:0; empty-cells:show }
    td, th { vertical-align:top; font-size:12pt;}
    h1, h2, h3, h4, h5, h6 { clear:both;}
    ol, ul { margin:0; padding:0;}
    li { list-style: none; margin:0; padding:0;}
    span.footnodeNumber { padding-right:1em; }
    span.annotation_style_by_filter { font-size:95%; font-family:Arial; background-color:#fff000;  margin:0; border:0; padding:0;  }
    span.heading_numbering { margin-right: 0.8rem; }* { margin:0;}
    .paragraph-Contents_20_1{ font-size:12pt; font-family:'Liberation Serif'; writing-mode:horizontal-tb; direction:ltr;margin-left:0cm; text-indent:0cm; }
    .paragraph-Contents_20_2{ font-size:12pt; font-family:'Liberation Serif'; writing-mode:horizontal-tb; direction:ltr;margin-left:0.499cm; text-indent:0cm; }
    .paragraph-Contents_20_3{ font-size:12pt; font-family:'Liberation Serif'; writing-mode:horizontal-tb; direction:ltr;margin-left:1cm; text-indent:0cm; }
    .paragraph-Contents_20_Heading{ font-size:16pt; font-weight:bold; margin-bottom:0.212cm; margin-left:0cm; margin-top:0.423cm; text-indent:0cm; font-family:'Liberation Sans'; writing-mode:horizontal-tb; direction:ltr;}
    .paragraph-Heading_20_2{ font-size:18pt; margin-bottom:0.212cm; margin-top:0.353cm; font-family:'Liberation Serif'; writing-mode:horizontal-tb; direction:ltr;font-weight:bold; }
    .paragraph-P1{ font-size:24pt; font-weight:bold; margin-bottom:0.247cm; margin-top:0cm; font-family:'Liberation Serif'; writing-mode:horizontal-tb; direction:ltr;line-height:115%; }
    .paragraph-P10{ font-size:12pt; line-height:115%; margin-bottom:0cm; margin-top:0cm; font-family:'Liberation Serif'; writing-mode:horizontal-tb; direction:ltr;margin-left:0cm; margin-right:0cm; text-indent:0cm; }
    .paragraph-P11{ font-size:10pt; margin-bottom:0.499cm; margin-top:0cm; font-family:monospace; writing-mode:horizontal-tb; direction:ltr;margin-left:0cm; margin-right:0cm; text-indent:0cm; }
    .paragraph-P12{ font-size:10pt; margin-bottom:0cm; margin-top:0cm; font-family:monospace; writing-mode:horizontal-tb; direction:ltr;margin-left:0cm; margin-right:0cm; text-indent:0cm; }
    .paragraph-P13{ font-size:10pt; margin-bottom:0cm; margin-top:0cm; font-family:monospace; writing-mode:horizontal-tb; direction:ltr;}
    .paragraph-P14{ font-size:10pt; margin-bottom:0.499cm; margin-top:0cm; font-family:monospace; writing-mode:horizontal-tb; direction:ltr;}
    .paragraph-P15{ font-size:10pt; margin-bottom:0.499cm; margin-top:0cm; font-family:monospace; writing-mode:horizontal-tb; direction:ltr;margin-left:0cm; margin-right:0cm; text-indent:0cm; }
    .paragraph-P16{ font-size:14pt; font-weight:bold; margin-bottom:0.212cm; margin-top:0.247cm; font-family:'Liberation Serif'; writing-mode:horizontal-tb; direction:ltr;margin-left:0cm; margin-right:0cm; text-indent:0cm; }
    .paragraph-P17{ font-size:12pt; line-height:115%; margin-bottom:0.247cm; margin-top:0cm; font-family:'Liberation Serif'; writing-mode:horizontal-tb; direction:ltr;}
    .paragraph-P18{ font-size:18pt; font-weight:bold; margin-bottom:0.212cm; margin-top:0.353cm; font-family:'Liberation Serif'; writing-mode:horizontal-tb; direction:ltr;margin-left:0cm; margin-right:0cm; text-indent:0cm; }
    .paragraph-P19{ font-size:12pt; line-height:115%; margin-bottom:0cm; margin-top:0cm; font-family:'Liberation Serif'; writing-mode:horizontal-tb; direction:ltr;}
    .paragraph-P2{ font-size:12pt; margin-left:0cm; text-indent:0cm; font-family:'Liberation Serif'; writing-mode:horizontal-tb; direction:ltr;}
    .paragraph-P20{ font-size:10pt; margin-bottom:0cm; margin-top:0cm; font-family:monospace; writing-mode:horizontal-tb; direction:ltr;margin-left:0cm; margin-right:0cm; text-indent:0cm; color:#666666; font-style:italic; }
    .paragraph-P21{ font-size:10pt; margin-bottom:0cm; margin-top:0cm; font-family:monospace; writing-mode:horizontal-tb; direction:ltr;margin-left:0cm; margin-right:0cm; text-indent:0cm; color:#000000; font-weight:bold; }
    .paragraph-P22{ font-size:10pt; margin-bottom:0cm; margin-top:0cm; font-family:'Liberation Mono'; writing-mode:horizontal-tb; direction:ltr;margin-left:0cm; margin-right:0cm; text-indent:0cm; }
    .paragraph-P23{ font-size:10pt; margin-bottom:0cm; margin-top:0cm; font-family:monospace; writing-mode:horizontal-tb; direction:ltr;color:#cc0000; font-style:italic; }
    .paragraph-P3{ font-size:12pt; margin-left:0.499cm; text-indent:0cm; font-family:'Liberation Serif'; writing-mode:horizontal-tb; direction:ltr;}
    .paragraph-P4{ font-size:12pt; margin-left:1cm; text-indent:0cm; font-family:'Liberation Serif'; writing-mode:horizontal-tb; direction:ltr;}
    .paragraph-P5{ font-size:12pt; line-height:115%; margin-bottom:0.247cm; margin-top:0cm; font-family:'Liberation Serif'; writing-mode:horizontal-tb; direction:ltr;margin-left:0cm; margin-right:0cm; text-indent:0cm; }
    .paragraph-P6{ font-size:18pt; font-weight:bold; margin-bottom:0.212cm; margin-top:0.353cm; font-family:'Liberation Serif'; writing-mode:horizontal-tb; direction:ltr;margin-left:0cm; margin-right:0cm; text-indent:0cm; }
    .paragraph-P7{ font-size:12pt; line-height:115%; margin-bottom:0cm; margin-top:0cm; font-family:'Liberation Serif'; writing-mode:horizontal-tb; direction:ltr;}
    .paragraph-P8{ font-size:10pt; margin-bottom:0.499cm; margin-top:0cm; font-family:'Liberation Mono'; writing-mode:horizontal-tb; direction:ltr;margin-left:0cm; margin-right:0cm; text-indent:0cm; }
    .paragraph-P9{ font-size:18pt; font-weight:bold; margin-bottom:0.212cm; margin-top:0.353cm; font-family:'Liberation Serif'; writing-mode:horizontal-tb; direction:ltr;margin-left:0cm; margin-right:0cm; text-indent:0cm; }
    .paragraph-Preformatted_20_Text{ font-size:10pt; font-family:'Liberation Mono'; writing-mode:horizontal-tb; direction:ltr;margin-top:0cm; margin-bottom:0cm; }
    .paragraph-Text_20_body{ font-size:12pt; font-family:'Liberation Serif'; writing-mode:horizontal-tb; direction:ltr;margin-top:0cm; margin-bottom:0.247cm; line-height:115%; }
    .section-Sect2{ margin-left:0cm; margin-right:0cm; }
    .text-Source_20_Text{ font-family:'Liberation Mono'; }
    .text-Strong_20_Emphasis{ font-weight:bold; }
    .text-T1{ text-decoration:underline; }
    .text-T10{ color:#007800; }
    .text-T11{ color:#000000; font-weight:bold; }
    .text-T12{ color:#7a0874; font-weight:bold; }
    .text-T13{ color:#660033; font-family:monospace; }
    .text-T14{ color:#000000; font-family:monospace; font-weight:bold; }
    .text-T15{ color:#007800; font-family:monospace; }
    .text-T16{ color:#ff0000; font-family:monospace; }
    .text-T17{ color:#7a0874; font-family:monospace; font-weight:bold; }
    .text-T18{ color:#000000; font-family:monospace; }
    .text-T19{ color:#cc0000; font-style:italic; }
    .text-T2{ font-family:monospace; }
    .text-T20{ color:#ff0000; }
    .text-T3{ color:#c20cb9; font-family:monospace; font-weight:bold; }
    .text-T4{ font-style:italic; }
    .text-T5{ color:#660033; }
    .text-T6{ color:#000000; }
    .text-T7{ color:#ff0000; }
    .text-T8{ color:#ff0000; }
    .text-T9{ color:#c20cb9; font-weight:bold; }
    /* ODF styles with no properties representable as CSS:
    .dp1 .Sect1 .Index_20_Link  { } */
</style>
</head>

<body>
<i><a href="https://www.linuxtricks.fr/wiki/openssl-creation-d-une-autorite-de-certification-interne-et-de-certificats-clients">Linuxtricks.fr - Openssl : Création d'une Autorité de Certification interne et de Certificats Clients</a></i>
<p class="paragraph-Contents_20_Heading">Table des matières</p>

<p class="paragraph-P2"><a href="#__RefHeading___Toc1508_2836107182" class="text-Index_20_Link">Openssl - Création d'une Autorité de Certification interne et de Certificats Clients</a></p>

<p class="paragraph-P3"><a href="#__RefHeading___Toc1510_2836107182" class="text-Index_20_Link">Introduction</a></p>

<p class="paragraph-P3"><a href="#__RefHeading___Toc1512_2836107182" class="text-Index_20_Link">Prérequis</a></p>

<p class="paragraph-P3"><a href="#__RefHeading___Toc1514_2836107182" class="text-Index_20_Link">Générer la CA</a></p>

<p class="paragraph-P3"><a href="#__RefHeading___Toc1516_2836107182" class="text-Index_20_Link">Déployer le CA sur les systèmes d'exploitation</a></p>

<p class="paragraph-P3"><a href="#__RefHeading___Toc1518_2836107182" class="text-Index_20_Link">Créer un certificat client</a></p>

<p class="paragraph-P3"><a href="#__RefHeading___Toc1520_2836107182" class="text-Index_20_Link">Mise en place des certificats</a></p>

<p class="paragraph-P4"><a href="#__RefHeading___Toc1522_2836107182" class="text-Index_20_Link">Apache dans le virtualhost SSL</a></p>

<p class="paragraph-P3"><a href="#__RefHeading___Toc1524_2836107182" class="text-Index_20_Link">Nginx dans le virtualhost SSL</a></p>

<p class="paragraph-P3"><a href="#__RefHeading___Toc1526_2836107182" class="text-Index_20_Link">Script de génération de certificat client</a></p>
<h2 class="paragraph-Heading_20_2"><a id="a__Introduction"><span/></a><a id="paragraph-introduction"/><a id="__RefHeading___Toc1510_2836107182"/>Introduction</h2>

<p class="paragraph-P5"><br/>Cet article décrit brièvement comment générer :<br/>- Une autorité de certification pour son usage perso<br/>- Un certificat SSL pour un domaine<br/>- Paramétrer les services concernés : Apache, Ngnix, ESXi<br/><br/>Au lieu de le faire valider par une autorité payante, on va créer notre autorité de certification personnelle.<br/><br/>Quelques abréviations utilisées dans l'article :<br/>- <span class="text-Strong_20_Emphasis">CA</span> : Certificate Authority, Autorité de Certification<br/>- <span class="text-Strong_20_Emphasis">CSR</span> : Certificate Signing Request, Demande de Signature de Certificat<br/>- <span class="text-Strong_20_Emphasis">CN</span> : Common Name, Nom Commun (nom pleinement qualifié dans le cas du certificat client)<br/><br/>Il sera nécessaire, pour que le certificat soit reconnu officiellement, d'ajouter dans le magasin de certificat du système, notre CA.<br/><br/><span class="text-Strong_20_Emphasis"><span class="text-T1">Note :</span></span> Cet article est rédigé pour générer des certificats pour un domaine interne home.arpa.<br/>Nous sommes dans le cadre d'un homelab, par conséquent :<br/>- on ne fera pas un truc super carré<br/>- on ne fera qu'une ROOT CA (SUB CA facultative)<br/></p>
<h2 class="paragraph-P6"><a id="a__Prérequis"><span/></a><a id="paragraph-prerequis"/><a id="__RefHeading___Toc1512_2836107182"/>Prérequis</h2>

<p class="paragraph-P7">On utilisera OpenSSL, donc celui-ci doit être installé.<br/>Normalement, il est déjà installé, sinon :<br/><br/>Debian, Ubuntu et dérivées :</p>

<!--Next 'div' was a 'text:section'.-->
<div class="section-Sect2" id="copy-code-1-content">
<p class="paragraph-P8"><span class="text-T2">apt </span><span class="text-T3">install</span><span class="text-T2"> openssl</span></p>
</div>

<p class="paragraph-P7">Fedora, Red Hat Enterprise Linux et dérivées :</p>

<!--Next 'div' was a 'text:section'.-->
<div class="section-Sect2" id="copy-code-2-content">
<p class="paragraph-P8"><span class="text-T2">dnf </span><span class="text-T3">install</span><span class="text-T2"> openssl</span></p>
</div>
<h2 class="paragraph-P9"><a id="a__Générer_la_CA"><span/></a><a id="paragraph-generer-la-ca"/><a id="__RefHeading___Toc1514_2836107182"/>Générer la CA </h2>

<p class="paragraph-P7">On a besoin d'un certificat racine d'abord.<br/><br/>On va créer une clé privée pour la CA <span class="text-Strong_20_Emphasis">homearpaCA.key</span> :</p>

<p class="paragraph-P10"><span class="text-T4"/></p>

<!--Next 'div' was a 'text:section'.-->
<div class="section-Sect2" id="copy-code-3-content">
<p class="paragraph-P11">openssl genpkey <span class="text-T5">-algorithm</span> RSA <span class="text-T5">-out</span> homearpaCA.key <span class="text-T5">-pkeyopt</span> rsa_keygen_bits:<span class="text-T6">4096</span> <span class="text-T5">-aes256</span></p>
</div>

<p class="paragraph-P7">Lors de la génération, il est demandé une passphrase :</p>

<p class="paragraph-P10"><span class="text-T4"/></p>

<!--Next 'div' was a 'text:section'.-->
<div class="section-Sect2" id="copy-code-4-content">
<p class="paragraph-P12">..+............+.......+........+.+..+.........+....+......................+++++</p>

<p class="paragraph-P13">.........</p>

<p class="paragraph-P13">Enter PEM pass phrase:</p>

<p class="paragraph-P14">Verifying - Enter PEM pass phrase:</p>
</div>

<p class="paragraph-P7">Maintenant qu'on a la clé, on va générer le certificat racine <span class="text-Strong_20_Emphasis">homearpaCA.pem</span> :</p>

<p class="paragraph-P10"><span class="text-T4"/></p>

<!--Next 'div' was a 'text:section'.-->
<div class="section-Sect2" id="copy-code-5-content">
<p class="paragraph-P15">openssl req <span class="text-T5">-x509</span> <span class="text-T5">-new</span> <span class="text-T5">-nodes</span> <span class="text-T5">-key</span> homearpaCA.key <span class="text-T5">-sha256</span> <span class="text-T5">-days</span> <span class="text-T6">7300</span> <span class="text-T5">-subj</span> <span class="text-T7">"/C=FR/ST=</span><span class="text-T8">PAYS-DE-LOIRE</span><span class="text-T7">/L=</span><span class="text-T8">ANGERS</span><span class="text-T7">/O=</span><span class="text-T8">HOMEARPA</span><span class="text-T7">/OU=</span><span class="text-T8">HOMEARPA</span><span class="text-T7">/CN= </span><span class="text-T8">HOMEARPA</span><span class="text-T7"> CA"</span> <span class="text-T5">-out</span> homearpaCA.pem</p>
</div>

<p class="paragraph-P10">Au moment de la génération, il est demandé la passphrase de la clé précédente.<br/><br/><span class="text-T1">Note :</span> Le <span class="text-Strong_20_Emphasis">-subj</span> permet d'éviter de répondre à un certain nombre de questions (pratique pour du non interractif). Ici, je génère le certificat pour une durée de 20 ans.<br/><br/>On va se générer un certificat au format <span class="text-Strong_20_Emphasis">crt</span> en plus du <span class="text-Strong_20_Emphasis">pem</span> pour la CA via :</p>

<p class="paragraph-P10"><span class="text-T4"/></p>

<!--Next 'div' was a 'text:section'.-->
<div class="section-Sect2" id="copy-code-6-content">
<p class="paragraph-P11">openssl x509 <span class="text-T5">-in</span> homearpaCA.pem <span class="text-T5">-inform</span> PEM <span class="text-T5">-out</span> homearpaCA.crt</p>
</div>

<p class="paragraph-Text_20_body"><br/><br/>Nous avons maintenant 3 fichiers :<br/>- <span class="text-Strong_20_Emphasis">homearpaCA.key</span> =&gt; La clé privée<br/>- <span class="text-Strong_20_Emphasis">homearpaCA.pem</span> =&gt; Certificat racine au format pem<br/>- <span class="text-Strong_20_Emphasis">homearpaCA.crt</span> =&gt; Certificat racine au format crt<br/></p>
<h2 class="paragraph-P9"><a id="a__Déployer_le_CA_sur_les_systèmes_d'exploitation"><span/></a><a id="paragraph-deployer-le-ca-sur-les-systemes-d-exploitation"/><a id="__RefHeading___Toc1516_2836107182"/>Déployer le CA sur les systèmes d'exploitation</h2>

<p class="paragraph-Text_20_body"><br/>Pour déployer cette autorité de certification, je vous renvoie aux articles suivants :<br/>- Red Hat, CentOS, Alma Linux, Rocky : Ajouter une autorité de certification au magasin système : <a href="https://www.linuxtricks.fr/wiki/red-hat-centos-alma-linux-rocky-ajouter-une-autorite-de-certification-au-magasin-systeme" class="text-Internet_20_link">https://www.linuxtricks.fr/wiki/red-hat-centos-alma-linux-rocky-ajouter-une-autorite-de-certification-au-magasin-systeme</a><br/>- Ubuntu : Ajouter une autorité de certification au magasin système : <a href="https://www.linuxtricks.fr/wiki/ubuntu-ajouter-une-autorite-de-certification-au-magasin-systeme" class="text-Internet_20_link">https://www.linuxtricks.fr/wiki/ubuntu-ajouter-une-autorite-de-certification-au-magasin-systeme</a><br/>- Firefox : Ajouter une autorité de certification au magasin local : <a href="https://www.linuxtricks.fr/wiki/firefox-ajouter-une-autorite-de-certification-au-magasin-local" class="text-Internet_20_link">https://www.linuxtricks.fr/wiki/firefox-ajouter-une-autorite-de-certification-au-magasin-local</a></p>
<h2 class="paragraph-P9"><a id="a__Créer_un_certificat_client"><span/></a><a id="paragraph-creer-un-certificat-client"/><a id="__RefHeading___Toc1518_2836107182"/>Créer un certificat client</h2>

<p class="paragraph-P7">Maintenant qu'on a un CA maison et installé sur nos hôtes, on va pouvoir générer des certificats clients pour nos serveurs (Sites Web, équipements, etc....) et les signer avec notre CA.<br/><br/>Dans cet exemple, le serveur concerné est <span class="text-Strong_20_Emphasis">cwwk.home.arpa</span><br/><br/>On va créer le fichier des directives de configuration du certificat :</p>

<p class="paragraph-P10"><span class="text-T4"/></p>

<!--Next 'div' was a 'text:section'.-->
<div class="section-Sect2" id="copy-code-7-content">
<p class="paragraph-P11"><span class="text-T9">nano</span> cwwk.home.arpa.ext</p>
</div>

<p class="paragraph-P7">On y place ceci :</p>
<p class="paragraph-P10"> </p>
<!--Next 'div' was a 'text:section'.-->
<div class="section-Sect2" id="copy-code-8-content">
<p class="paragraph-P12"><span class="text-T10">authorityKeyIdentifier</span>=keyid,issuer</p>

<p class="paragraph-P12"><span class="text-T10">basicConstraints</span>=CA:FALSE</p>

<p class="paragraph-P13">keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment</p>

<p class="paragraph-P12">subjectAltName = <span class="text-T11">@</span>alt_names</p>

<p class="paragraph-P12"><span class="text-T12">[</span>alt_names<span class="text-T12">]</span></p>

<p class="paragraph-P14">DNS.1 = cwwk.home.arpa</p>
</div>

<p class="paragraph-P7">On crée d'abord la clé privée :</p>

<p class="paragraph-P10"><span class="text-T4"/></p>

<!--Next 'div' was a 'text:section'.-->
<div class="section-Sect2" id="copy-code-9-content">
<p class="paragraph-P11">openssl genpkey <span class="text-T5">-algorithm</span> RSA <span class="text-T5">-out</span> cwwk.home.arpa.key <span class="text-T5">-pkeyopt</span> rsa_keygen_bits:<span class="text-T6">4096</span></p>
</div>

<p class="paragraph-P7">Ensuite il faut générer la demande de signature de certificat :</p>

<p class="paragraph-P10"><span class="text-T4"/></p>

<!--Next 'div' was a 'text:section'.-->
<div class="section-Sect2" id="copy-code-10-content">
<p class="paragraph-P11">openssl req <span class="text-T5">-new</span> <span class="text-T5">-key</span> cwwk.home.arpa.key <span class="text-T5">-out</span> cwwk.home.arpa.csr <span class="text-T5">-subj</span> <span class="text-T7">"/C=FR/ST=BOURGOGNE/L=DIJON/O=homearpa/OU=homearpa/CN=cwwk.home.arpa"</span></p>
</div>

<p class="paragraph-P10"><span class="text-T1">Note :</span> Le <span class="text-Strong_20_Emphasis">-subj</span> permet d'éviter de répondre à un certain nombre de questions (pratique pour du non interractif).<br/><span class="text-Strong_20_Emphasis">Il faut bien mettre le nom du serveur tel qu'il est appelé de l'extérieur dans le champ CN !</span><br/><br/>Ensuite on signe la CSR avec la CA :</p>

<p class="paragraph-P10"><span class="text-T4"/></p>

<!--Next 'div' was a 'text:section'.-->
<div class="section-Sect2" id="copy-code-11-content">
<p class="paragraph-P11">openssl x509 <span class="text-T5">-req</span> <span class="text-T5">-in</span> cwwk.home.arpa.csr <span class="text-T5">-CA</span> homearpaCA.crt <span class="text-T5">-CAkey</span> homearpaCA.key <span class="text-T5">-CAcreateserial</span> <span class="text-T5">-out</span> cwwk.home.arpa.crt <span class="text-T5">-days</span> <span class="text-T6">3650</span> <span class="text-T5">-sha256</span> <span class="text-T5">-extfile</span> cwwk.home.arpa.ext</p>
</div>

<p class="paragraph-P7">On peut vérifier que le certificat est bien signé via :</p>

<p class="paragraph-P10"><span class="text-T4"/></p>

<!--Next 'div' was a 'text:section'.-->
<div class="section-Sect2" id="copy-code-12-content">
<p class="paragraph-P11">openssl verify <span class="text-T5">-CAfile</span> homearpaCA.crt cwwk.home.arpa.crt</p>
</div>

<p class="paragraph-P7">Ce qui renvoie :</p>

<p class="paragraph-P10"><a id="copy-code-13"/><span class="text-T4">Copier vers le presse-papier</span>Code :</p>

<!--Next 'div' was a 'text:section'.-->
<div class="section-Sect2" id="copy-code-13-content">
<p class="paragraph-P10"><span class="text-Source_20_Text"><span class="text-T6">cwwk.home.arpa.crt: OK</span></span><span class="text-Source_20_Text"> </span></p>
</div>

<p class="paragraph-Text_20_body"><br/>Le certificat généré sera valide 10 ans dans mon exemple.<br/><br/>Nous avons maintenant 4 fichiers :<br/>- <span class="text-Strong_20_Emphasis">cwwk.home.arpa.key</span> =&gt; La clé privée du certificat client<br/>- <span class="text-Strong_20_Emphasis">cwwk.home.arpa.csr</span> =&gt; La demande de signature du certificat client<br/>- <span class="text-Strong_20_Emphasis">cwwk.home.arpa.crt</span> =&gt; Certificat client au format crt<br/>- <span class="text-Strong_20_Emphasis">cwwk.home.arpa.ext</span> =&gt; Directives de configuration des certificats<br/><br/>Seuls les fichiers <span class="text-Strong_20_Emphasis">.key</span> et <span class="text-Strong_20_Emphasis">.crt</span> sont utiles par la suite.</p>
<h2 class="paragraph-P9"><a id="a__Mise_en_place_des_certificats"><span/></a><a id="paragraph-mise-en-place-des-certificats"/><a id="__RefHeading___Toc1520_2836107182"/>Mise en place des certificats</h2>
<h3 class="paragraph-P16"><a id="a__Apache_dans_le_virtualhost_SSL"><span/></a><a id="paragraph-apache-dans-le-virtualhost-ssl"/><a id="__RefHeading___Toc1522_2836107182"/>Apache dans le virtualhost SSL</h3>

<p class="paragraph-P7">Envoyer les fichiers de certificat (CRT) et Clé (KEY).<br/><br/><span class="text-Strong_20_Emphasis">Si on utilise SELinux</span>, on mettra le contexte adéquat :</p>

<p class="paragraph-P10"><span class="text-T4"/></p>

<!--Next 'div' was a 'text:section'.-->
<div class="section-Sect2" id="copy-code-14-content">
<p class="paragraph-P11">chcon <span class="text-T5">-R</span> <span class="text-T5">-t</span> cert_t <span class="text-T11">/</span>etc<span class="text-T11">/</span>ssl<span class="text-T11">/</span>private<span class="text-T11">/</span></p>
</div>

<p class="paragraph-Text_20_body">Éditer les lignes suivantes et renseigner les chemins des fichiers de certificat dans la configuration du VirtualHost :</p>

<!--Next 'div' was a 'text:section'.-->
<div class="section-Sect2" id="copy-code-15-content">
<p class="paragraph-P12">ServerName homearpa.fr</p>

<p class="paragraph-P12">SSLCertificateFile <span class="text-T11">/</span>etc<span class="text-T11">/</span>ssl<span class="text-T11">/</span>private<span class="text-T11">/</span>cwwk.home.arpa.crt</p>

<p class="paragraph-P11">SSLCertificateKeyFile <span class="text-T11">/</span>etc<span class="text-T11">/</span>ssl<span class="text-T11">/</span>private<span class="text-T11">/</span>cwwk.home.arpa.key</p>
</div>

<p class="paragraph-Text_20_body">Recharger le démon apache2 (ou httpd suivant le système).<br/></p>
<h2 class="paragraph-P9"><a id="a__Nginx_dans_le_virtualhost_SSL"><span/></a><a id="paragraph-nginx-dans-le-virtualhost-ssl"/><a id="__RefHeading___Toc1524_2836107182"/>Nginx dans le virtualhost SSL</h2>

<p class="paragraph-P7">Envoyer les fichiers de certificat (CRT) et Clé (KEY).<br/><span class="text-Strong_20_Emphasis">Si on utilise SELinux</span>, on mettra le contexte adéquat :</p>

<!--Next 'div' was a 'text:section'.-->
<div class="section-Sect2" id="copy-code-16-content">
<p class="paragraph-P8"><span class="text-T2">chcon </span><span class="text-T13">-R</span><span class="text-T2"> </span><span class="text-T13">-t</span><span class="text-T2"> cert_t </span><span class="text-T14">/</span><span class="text-T2">etc</span><span class="text-T14">/</span><span class="text-T2">ssl</span><span class="text-T14">/</span><span class="text-T2">private</span><span class="text-T14">/</span></p>
</div>

<p class="paragraph-Text_20_body">Éditer les lignes suivantes et renseigner les chemins des fichiers de certificat dans la configuration du VirtualHost :</p>

<!--Next 'div' was a 'text:section'.-->
<div class="section-Sect2" id="copy-code-17-content">
<p class="paragraph-P12">ssl                  on;</p>

<p class="paragraph-P12">ssl_certificate      <span class="text-T11">/</span>etc<span class="text-T11">/</span>ssl<span class="text-T11">/</span>private<span class="text-T11">/</span>cwwk.home.arpa.crt;</p>

<p class="paragraph-P11">ssl_certificate_key  <span class="text-T11">/</span>etc<span class="text-T11">/</span>ssl<span class="text-T11">/</span>private<span class="text-T11">/</span>cwwk.home.arpa.key;</p>
</div>

<p class="paragraph-P17">Recharger le démon nginx.<br/></p>
<h2 class="paragraph-P18"><a id="a__Script_de_génération_de_certificat_client"><span/></a><a id="paragraph-script-de-generation-de-certificat-client"/><a id="__RefHeading___Toc1526_2836107182"/>Script de génération de certificat client</h2>

<p class="paragraph-P7">Comme dans mon lab je génère des certificats client de temps en temps, je ne retiens pas les commandes.<br/>Voici l'arborescence :</p>

<p class="paragraph-P10"><span class="text-T4"/></p>

<!--Next 'div' was a 'text:section'.-->
<div class="section-Sect2" id="copy-code-18-content">
<p class="paragraph-P12">CA/</p>

<p class="paragraph-Preformatted_20_Text">   <span class="text-T2">homearpaCA.crt</span></p>

<p class="paragraph-Preformatted_20_Text">   <span class="text-T2">homearpaCA.info</span></p>

<p class="paragraph-Preformatted_20_Text">   <span class="text-T2">homearpaCA.key</span></p>

<p class="paragraph-Preformatted_20_Text">   <span class="text-T2">homearpaCA.pem</span></p>

<p class="paragraph-Preformatted_20_Text">   <span class="text-T2">homearpaCA.srl</span></p>

<p class="paragraph-P14">gencert.sh</p>
</div>

<p class="paragraph-P19">Voici le script très simpliste :</p>
<p class="paragraph-P10"> </p>
<!--Next 'div' was a 'text:section'.-->
<div class="section-Sect2" id="copy-code-19-content">
<p class="paragraph-P20">#! /bin/bash</p>

<p class="paragraph-P20"># Chemin des fichiers du CA </p>

<p class="paragraph-P20"># Si dans CA on a les homearpaCA.key homearpaCA.crt homearpaCA.ext : CA="CA/homearpaCA"</p>

<p class="paragraph-P12"><span class="text-T10">CA</span>=<span class="text-T7">"CA/homearpaCA"</span></p>

<p class="paragraph-P20"># Domaine du CA</p>

<p class="paragraph-P12"><span class="text-T10">DOMAINE</span>=<span class="text-T7">"home.arpa"</span></p>

<p class="paragraph-P20">################</p>

<p class="paragraph-P20"># DEBUT SCRIPT #</p>

<p class="paragraph-P20">################</p>

<p class="paragraph-P12"><span class="text-T10">DOM1</span>=<span class="text-T7">"</span><span class="text-T10">${DOMAINE%%.*}</span><span class="text-T7">"</span></p>

<p class="paragraph-P12"><span class="text-T10">DOM2</span>=<span class="text-T7">"</span><span class="text-T10">${DOMAINE#*.}</span><span class="text-T7">"</span></p>

<p class="paragraph-P12"><span class="text-T11">if</span> <span class="text-T12">[[</span> <span class="text-T7">"$1"</span> =~ ^<span class="text-T12">[</span>a-zA-Z0-<span class="text-T6">9</span>-<span class="text-T12">]</span>+\.<span class="text-T10">$DOM1</span>\.<span class="text-T10">$DOM2</span> <span class="text-T12">]]</span></p>

<p class="paragraph-P21">then</p>

<p class="paragraph-P22">    <span class="text-T15">FQDN</span><span class="text-T2">=</span><span class="text-T16">"$1"</span></p>

<p class="paragraph-P21">else</p>

<p class="paragraph-P22">    <span class="text-T17">echo</span><span class="text-T2"> </span><span class="text-T16">"Erreur : FQDN $1 incorrect pour le domaine </span><span class="text-T15">$DOMAINE</span><span class="text-T16">"</span></p>

<p class="paragraph-P22">    <span class="text-T17">exit</span><span class="text-T2"> </span><span class="text-T18">1</span></p>

<p class="paragraph-P21">fi</p>

<p class="paragraph-P12"><span class="text-T12">echo</span> <span class="text-T7">"Certificat pour : </span><span class="text-T10">$FQDN</span><span class="text-T7">"</span></p>

<p class="paragraph-P12"><span class="text-T12">echo</span> <span class="text-T7">"Fichier de directives de configuration"</span></p>

<p class="paragraph-P12"><span class="text-T9">cat</span> <span class="text-T11">&gt;</span> <span class="text-T10">$FQDN</span>.ext <span class="text-T19">&lt;&lt; EOF</span></p>

<p class="paragraph-P23">authorityKeyIdentifier=keyid,issuer</p>

<p class="paragraph-P23">basicConstraints=CA:FALSE</p>

<p class="paragraph-P23">keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment</p>

<p class="paragraph-P23">subjectAltName = @alt_names</p>

<p class="paragraph-P23">[alt_names]</p>

<p class="paragraph-P23">DNS.1 = $FQDN</p>

<p class="paragraph-P23">EOF</p>

<p class="paragraph-P12"><span class="text-T12">echo</span> <span class="text-T7">"Generation KEY"</span></p>

<p class="paragraph-P12">openssl genpkey <span class="text-T5">-algorithm</span> RSA <span class="text-T5">-out</span> <span class="text-T10">$FQDN</span>.key <span class="text-T5">-pkeyopt</span> rsa_keygen_bits:<span class="text-T6">4096</span></p>

<p class="paragraph-P12"><span class="text-T12">echo</span> <span class="text-T7">"Génération CSR"</span></p>

<p class="paragraph-P12">openssl req <span class="text-T5">-new</span> <span class="text-T5">-key</span> <span class="text-T10">$FQDN</span>.key <span class="text-T5">-out</span> <span class="text-T10">$FQDN</span>.csr <span class="text-T5">-subj</span> <span class="text-T7">"/C=FR/ST=</span><span class="text-T20">PAYS-DE-LOIRE</span><span class="text-T7">/L=</span><span class="text-T20">ANGERS</span><span class="text-T7">/O=homearpa/OU=homearpa/CN=</span><span class="text-T10">$FQDN</span><span class="text-T7">"</span></p>

<p class="paragraph-P12"><span class="text-T12">echo</span> <span class="text-T7">"Signature CSR avec CA"</span></p>

<p class="paragraph-P12">openssl x509 <span class="text-T5">-req</span> <span class="text-T5">-in</span> <span class="text-T10">$FQDN</span>.csr <span class="text-T5">-CA</span> <span class="text-T10">$CA</span>.crt <span class="text-T5">-CAkey</span> <span class="text-T10">$CA</span>.key <span class="text-T5">-CAcreateserial</span> <span class="text-T5">-out</span> <span class="text-T10">$FQDN</span>.crt <span class="text-T5">-days</span> <span class="text-T6">3650</span> <span class="text-T5">-sha256</span> <span class="text-T5">-extfile</span> <span class="text-T10">$FQDN</span>.ext</p>

<p class="paragraph-P12"><span class="text-T12">echo</span> <span class="text-T7">"Vérification"</span></p>

<p class="paragraph-P8"><span class="text-T2">openssl verify </span><span class="text-T13">-CAfile</span><span class="text-T2"> </span><span class="text-T15">$CA</span><span class="text-T2">.crt </span><span class="text-T15">$FQDN</span><span class="text-T2">.crt</span></p>
<p class="paragraph-P11"> </p></div>
</body>

</html>
