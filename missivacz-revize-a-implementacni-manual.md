# missiva.cz - Revize a implementační manuál

# Roztříštěnost kódů

Část kódů je napevno v kódu a část je realizována prostřednictvím Google Tag Manageru.

U Facebook měření je dokonce měření do jednoho pixelu umístěno v kódu a měření do druhého pixelu je v GTM.

Důsledkem toho je **duplikace událostí odesílaných do Facebooku**, přičemž každá z těchto událostí má mírně odlišné parametry.

To ve výsledku může způsobovat **významné odchylky mezi reálnými daty a daty ve Facebooku**. Optimalizace kampaní na základě těchto dat je poté neefektivní.

# Nevhodná práce se souhlasy

![image-20251118-174928.png](https://zdenekhejl.atlassian.net/wiki/download/attachments/64159745/image-20251118-174928.png?api=v2)

## Nevhodné seskupení souhlasů

Nyní jsou souhlasy seskupené následujícím způsobem:

- **Analytické souhlasy** \= ad\_storage a analytics\_storage
- **Marketingové souhlasy** = ad\_user\_data a ad\_personalization a personalization\_storage

```
<script type="text/plain" data-lib-cookieconsent="performance">
    gtag('consent', 'update', { 'ad_storage': 'granted', 'analytics_storage': 'granted' });
</script>

<script type="text/plain" data-lib-cookieconsent="marketing">
    gtag('consent', 'update', { 'ad_user_data': 'granted', 'ad_personalization': 'granted', 'personalization_storage': 'granted' });
</script>
```

Avšak ad\_storage nepatří mezi analytické souhlasy. **Kód by měl tedy být upravený následovně.**

```
<script type="text/plain" data-lib-cookieconsent="performance">
    gtag('consent', 'update', { 'analytics_storage': 'granted' });
</script>

<script type="text/plain" data-lib-cookieconsent="marketing">
    gtag('consent', 'update', { 'ad_storage': 'granted', 'ad_user_data': 'granted', 'ad_personalization': 'granted', 'personalization_storage': 'granted' });
</script>
```

## Chybějící informace o update souhlasů v dataLayeru

Po změně provedené v předchozím kroku budou sice souhlasy již správně seskupeny, ale stále nebude možné v rámci GTM snadno odchytávat informace o update souhlasů.

Proto doporučuji **přidat k jednotlivým updatům taky odeslání informace o uděleném souhlasu do dataLayeru**.

Doplněný kód by poté vypadal následovně.

```
<script type="text/plain" data-lib-cookieconsent="performance">
    gtag('consent', 'update', { 'analytics_storage': 'granted' });
    window.dataLayer.push({'event': 'analytics_consent'});
</script>

<script type="text/plain" data-lib-cookieconsent="marketing">
    gtag('consent', 'update', { 'ad_storage': 'granted', 'ad_user_data': 'granted', 'ad_personalization': 'granted', 'personalization_storage': 'granted' });
    window.dataLayer.push({'event': 'marketing_consent'});
</script>
```

## Nevhodné pořadí kódů v šabloně stránky

Zatímco kód pro nastavení defaultních souhlasů je umístěný před GTM kódem, tak **updaty souhlasů jsou umístěné až za GTM kódem**.

Kvůli tomu nemá GTM při udělených souhlasech zavčas informaci o tom, že souhlasy byly uděleny.

Doporučuji přemístit kódy pro update souhlasů před GTM kód.

Tj. tento stávající kód

```
<script>
    window.dataLayer = window.dataLayer || [];
    function gtag(){ window.dataLayer.push(arguments) }

    gtag('consent', 'default', { 'ad_storage': 'denied', 'analytics_storage': 'denied', 'ad_user_data': 'denied', 'ad_personalization': 'denied', 'functionality_storage': 'granted', 'personalization_storage': 'denied', 'security_storage': 'granted' });
</script>
<!-- Google Tag Manager -->
<script>
  (function(w,d,s,l,i){ w[l]=w[l]||[]; w[l].push({'gtm.start':
  new Date().getTime(),event:'gtm.js'});var f=d.getElementsByTagName(s)[0],
  j=d.createElement(s),dl=l!='dataLayer'?'&l='+l:'';j.async=true;j.src=
  'https://www.googletagmanager.com/gtm.js?id='+i+dl;f.parentNode.insertBefore(j,f);
  })(window,document,'script','dataLayer',"GTM-KL38D93M");
</script>
<script type="text/plain" data-lib-cookieconsent="performance">
    gtag('consent', 'update', { 'ad_storage': 'granted', 'analytics_storage': 'granted' });
</script>

<script type="text/plain" data-lib-cookieconsent="marketing">
    gtag('consent', 'update', { 'ad_user_data': 'granted', 'ad_personalization': 'granted', 'personalization_storage': 'granted' });
</script>
```

by **po všech výše uvedených úpravách** (oprava seskupení souhlasů, odeslání informace o udělelné souhlasu do dataLayeru, přesun updatů souhlasů před GTM) **měl vypadat následovně**.

```
<script>
    window.dataLayer = window.dataLayer || [];
    function gtag(){ window.dataLayer.push(arguments) }

    gtag('consent', 'default', { 'ad_storage': 'denied', 'analytics_storage': 'denied', 'ad_user_data': 'denied', 'ad_personalization': 'denied', 'functionality_storage': 'granted', 'personalization_storage': 'denied', 'security_storage': 'granted' });
</script>

<script type="text/plain" data-lib-cookieconsent="performance">
    gtag('consent', 'update', { 'analytics_storage': 'granted' });
    window.dataLayer.push({'event': 'analytics_consent'});
</script>

<script type="text/plain" data-lib-cookieconsent="marketing">
    gtag('consent', 'update', { 'ad_storage': 'granted', 'ad_user_data': 'granted', 'ad_personalization': 'granted', 'personalization_storage': 'granted' });
    window.dataLayer.push({'event': 'marketing_consent'});
</script>

<!-- Google Tag Manager -->
<script>
  (function(w,d,s,l,i){ w[l]=w[l]||[]; w[l].push({'gtm.start':
  new Date().getTime(),event:'gtm.js'});var f=d.getElementsByTagName(s)[0],
  j=d.createElement(s),dl=l!='dataLayer'?'&l='+l:'';j.async=true;j.src=
  'https://www.googletagmanager.com/gtm.js?id='+i+dl;f.parentNode.insertBefore(j,f);
  })(window,document,'script','dataLayer',"GTM-KL38D93M");
</script>
```

# Odebrání Facebook kódů

Všechny Facebook kódy, které jsou aktuálně umístěné v šablonách stránek by **měly být odstraněny, protože nyní kolidují s Facebook měřením realizovaným přes GTM**.

Jedná se o základní Facebook kód, který je na každé stránce.

```
<!-- Facebook Tracking Code -->
<script type="text/plain" data-lib-cookieconsent="performance">
  !function(f,b,e,v,n,t,s){ if(f.fbq)return;n=f.fbq=function(){ n.callMethod?
          n.callMethod.apply(n,arguments):n.queue.push(arguments)};
          if(!f._fbq)f._fbq=n;n.push=n;n.loaded=!0;n.version='2.0';
          n.queue=[];t=b.createElement(e);t.async=!0;
          t.src=v;s=b.getElementsByTagName(e)[0];
          s.parentNode.insertBefore(t,s)}(window, document,'script',
    'https://connect.facebook.net/en_US/fbevents.js');
  fbq('init', "1547537389472851");
  fbq('track', 'PageView');
</script>
```

Tak také o dílčí kódy, které odesílají jednotlivé události do Facebooku.

```
<!-- Facebook event -->
<script type="text/plain" data-lib-cookieconsent="performance">
    fbq('track', "ViewContent", {"content_name":"Těšík Harmony 0,5l","content_category":"Čistící prostředky","content_ids":["1110"],"content_type":"product","value":309,"currency":"CZK"});
</script>

nebo

<!-- Facebook event -->
<script type="text/plain" data-lib-cookieconsent="performance">
    fbq('track', "ViewContent", {"content_name":"Permon J Sport 0,5l","content_category":"Prací program","content_ids":["1254"],"content_type":"product","value":150,"currency":"CZK"});
</script>

a další
```

Nově bude měření do Facebooku realizováno čistě prostřednictvím GTM.

# Odebrání omezení přidávání dat do dataLayeru

Informace o událostech na webu (zobrazení stránky, přidání do košíku, odeslaná objednávka apod.) se nyní odesílají do dataLayeru pouze v případě, že jsou dostupné analytické souhlasy.

![image-20251119-092757.png](https://zdenekhejl.atlassian.net/wiki/download/attachments/64159745/image-20251119-092757.png?api=v2)

Je potřeba, aby se tato **data odesílala do dataLayeru i bez dostupných souhlasů**.