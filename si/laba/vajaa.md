# 10. Vaja: Elektronska pošta

## Navodila

1. Oglejte si podrobnosti poljubnega elektronskega sporočila.
2. Pošljite elektronsko sporočilo, tako da se pretvarjate za nekoga drugega.

## Dodatne informacije

## Podrobna navodila

### 1. Podrobnosti elektronskih sporočil

Elektronska sporočila so poslana in prejete preko protokola [Simple Mail Transfer Protocol - SNMP](https://en.wikipedia.org/wiki/Simple_Mail_Transfer_Protocol)

Tradicionalni sistemi Unix hranijo e-pošto v formatu [`mbox`](https://en.wikipedia.org/wiki/Mbox), ki preprosto naniza sporočila v eno datoteko. Novejša možnost je [`maildir`](https://en.wikipedia.org/wiki/Maildir), ki vsako sporočilo hrani v ločeni datoteki. Precej uporabniških programov uporablja razne izpeljanke teh formatov, medtem ko drugi (npr. Windows Mail in Thunderbird) uporabljajo zapis [`eml`](https://en.wikipedia.org/wiki/Email#Filename_extensions). Mnogi programi hranijo še indeks za hitrejše in strukturirano iskanje.

Namesto prenosa sporočil na lokalni računalnik (s protokolom [POP3](https://en.wikipedia.org/wiki/Post_Office_Protocol)) vse več uporabnikov hrani pošto na strežniku in do nje dostopa s protokolom [IMAP](https://en.wikipedia.org/wiki/Internet_Message_Access_Protocol) oziroma spletnim odjemalcem.

V obeh primerih lahko poslana in prejeta sporočila zbrišemo ali spremenimo. Na svojem računalniku enostavno uredimo datoteko, pošto na strežniku pa lahko zamenjamo z [ukazi IMAP](https://www-archive.mozilla.org/quality/mailnews/tests/sea-mn-imap-commands) ([IMAP 101](https://www.atmail.com/blog/imap-101-manual-imap-sessions/), [Advanced IMAP](https://www.atmail.com/blog/advanced-imap/) in [IMAP Commands](https://www.atmail.com/blog/imap-commands/)).

Izberite poljubno elektronsko pošto z vsaj eno priponko ali prenesite primer s [tukaj](https://ucilnica.fri.uni-lj.si/mod/resource/view.php?id=52478). Vrstice od prve do zadnje pojavitve besede `Received` nam predstavljajo SMTP protokol, od besede `From` naprej sledi glava elektronskega sporočila in vsaka beseda `Content-Type` nam predstavlja vsebino.

    less example.xml

    Received: from VI1PR10MB3744.EURPRD10.PROD.OUTLOOK.COM (2603:10a6:803:135::22)
    by VI1PR10MB3309.EURPRD10.PROD.OUTLOOK.COM with HTTPS; Fri, 14 Apr 2023
    03:35:06 +0000
    Authentication-Results: dkim=none (message not signed)
    header.d=none;dmarc=none action=none header.from=fri.uni-lj.si;
    Received: from VI1PR10MB3584.EURPRD10.PROD.OUTLOOK.COM (2603:10a6:800:137::9)
    by VI1PR10MB3744.EURPRD10.PROD.OUTLOOK.COM (2603:10a6:803:135::22) with
    Microsoft SMTP Server (version=TLS1_2,
    cipher=TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384) id 15.20.6319.8; Fri, 14 Apr
    2023 03:35:04 +0000
    Received: from VI1PR10MB3584.EURPRD10.PROD.OUTLOOK.COM
    ([fe80::f9fe:a14b:f314:c186]) by VI1PR10MB3584.EURPRD10.PROD.OUTLOOK.COM
    ([fe80::f9fe:a14b:f314:c186%3]) with mapi id 15.20.6319.008; Fri, 14 Apr 2023
    03:35:04 +0000
    From: David Jelenc <david.jelenc@fri.uni-lj.si>
    To: Aleks =?utf-8?Q?Hu=C4=8D?= FRI <aleks.huc@fri.uni-lj.si>
    Subject: Man in Delorean
    Date: Fri, 14 Apr 2023 05:34:05 +0200
    User-agent: mu4e 1.8.10; emacs 28.1
    Message-ID: <87wn2fjfvh.fsf@fri.uni-lj.si>
    Content-Type: multipart/mixed; boundary="=-=-="
    X-MS-Exchange-Organization-ExpirationStartTime: 14 Apr 2023 03:35:01.6320
    (UTC)
    X-MS-Exchange-Organization-ExpirationStartTimeReason: OriginalSubmit
    X-MS-Exchange-Organization-ExpirationInterval: 1:00:00:00.0000000
    X-MS-Exchange-Organization-ExpirationIntervalReason: OriginalSubmit
    X-MS-Exchange-Organization-Network-Message-Id:
    3ba215ac-7bfb-4aa4-10ac-08db3c993abf
    X-MS-Exchange-Organization-AuthSource: VI1PR10MB3584.EURPRD10.PROD.OUTLOOK.COM
    X-MS-Exchange-Organization-AuthAs: Internal
    X-MS-Exchange-Organization-AuthMechanism: 06
    X-ClientProxiedBy: VI1PR07CA0260.eurprd07.prod.outlook.com
    (2603:10a6:803:b4::27) To VI1PR10MB3584.EURPRD10.PROD.OUTLOOK.COM
    (2603:10a6:800:137::9)
    X-MS-Exchange-Organization-MessageDirectionality: Originating
    Return-Path: David.Jelenc@fri.uni-lj.si
    X-MS-PublicTrafficType: Email
    X-MS-TrafficTypeDiagnostic:
    VI1PR10MB3584:EE_|VI1PR10MB3744:EE_|VI1PR10MB3309:EE_
    X-MS-Office365-Filtering-Correlation-Id: 3ba215ac-7bfb-4aa4-10ac-08db3c993abf
    X-MS-Exchange-Organization-SCL: -1
    X-Microsoft-Antispam: BCL:0;
    X-Forefront-Antispam-Report:
    CIP:255.255.255.255;CTRY:;LANG:en;SCL:-1;SRV:;IPV:NLI;SFV:SKI;H:VI1PR10MB3584.EURPRD10.PROD.OUTLOOK.COM;PTR:;CAT:NONE;SFS:;D
    IR:INT;
    X-MS-Exchange-CrossTenant-Network-Message-Id: 3ba215ac-7bfb-4aa4-10ac-08db3c993abf
    X-MS-Exchange-CrossTenant-AuthSource: VI1PR10MB3584.EURPRD10.PROD.OUTLOOK.COM
    X-MS-Exchange-CrossTenant-AuthAs: Internal
    X-MS-Exchange-CrossTenant-OriginalArrivalTime: 14 Apr 2023 03:35:03.5041
    (UTC)
    X-MS-Exchange-CrossTenant-FromEntityHeader: Hosted
    X-MS-Exchange-CrossTenant-Id: a6cc90df-f580-49dc-903f-87af5a75338e
    X-MS-Exchange-CrossTenant-MailboxType: HOSTED
    X-MS-Exchange-CrossTenant-UserPrincipalName: ZVyANDguTTml/VlroYVlk5dGCQFvTkUc7ErFaOm1If4DJu7ivoB7VoSVLlpGHn/UUDPSboZGXo81oKK2
    x95L0gJpLrXBBWLIszNxHDbQ3q0=
    X-MS-Exchange-Transport-CrossTenantHeadersStamped: VI1PR10MB3744
    X-MS-Exchange-Transport-EndToEndLatency: 00:00:02.6802416
    X-MS-Exchange-Processed-By-BccFoldering: 15.20.6319.007
    X-Microsoft-Antispam-Mailbox-Delivery:
        ucf:0;jmr:0;auth:0;dest:I;ENG:(910001)(944506478)(944626604)(920097)(425001)(930097);
    X-Microsoft-Antispam-Message-Info:
            ueXomgLddmbVL6JL2aHkXthoXpBYBxA74YRxKt+VsplU3bF0yePxsZlslzrB/d70EbDKZtdRJf0x4ADnYaDrRmsKPzFBZUveD/hac8RwQwVd+4DQdWfDWmZsgOaIsGy/blGz00HHnEZEm782V8+fEEAQ3jb0kSjQeIKeyHTQtRoLKFlgAprqubFFb4SBjvia+yZsEhp9Ph7RJhrg6vlK/cZLoHMB+CtbedZkuoOwm2ae7K7EPLGeaKmDBIPP6xGUCmeeZ9XaHSdUHMEDednte6/yaLiACW37kRgdNagK5RePQkXpdiALp6isWWp4W8LgPtDL8Y6AHwTWHjHth5mKvSUObUSYSCbu7xufhQh6i8WzowERBJFCJyJv5T1COKxG
    MIME-Version: 1.0

    --=-=-=
    Content-Type: multipart/alternative; boundary="==-=-="

    --==-=-=
    Content-Type: text/plain
    Content-Disposition: inline


    --==-=-=
    Content-Type: text/html
    Content-Disposition: inline

    <html xmlns="http://www.w3.org/1999/xhtml" lang="en" xml:lang="en"><head><!-- 2023-04-14 pet 05:34 --><meta http-equiv="Content-Type" content="text/html;charset=utf-8"/><meta name="viewport" content="width=device-width, initial-scale=1"/><meta name="generator" content="Org Mode"/></head><body>
    <div style="font-family:-apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, Roboto, Oxygen, Ubuntu, Cantarell,        &quot;Fira Sans&quot;, &quot;Droid Sans&quot;, &quot;Helvetica Neue&quot;, Arial, sans-serif, &quot;Apple Color Emoji&quot;, &quot;Segoe UI Emoji&quot;, &quot;Segoe UI Symbol&quot;;;font-size:11pt;line-height:12pt;" id="content">
    </div>
    </body></html>
    --==-=-=--

    --=-=-=
    Content-Type: image/jpeg
    Content-Disposition: attachment;
    filename=340993788_977418989868069_7643063285590221799_n.jpg
    Content-Transfer-Encoding: base64

    /9j/4AAQSkZJRgABAQAAAQABAAD/7QCEUGhvdG9zaG9wIDMuMAA4QklNBAQAAAAAAGgcAigAYkZC

    ...

    JeYA/k4KxPo9+IU/c+J9c/ifXP4n1z+JedN+lRPoc9S9WGp+p+p//9k=

    --=-=-=--

Sedaj poskusimo izluščiti prilogo iz elektronskega sporočila v samostojno datoteko in jo odpremo.

# 2. Pošiljanje elektronske pošte s ponarejenimi metapodatki

