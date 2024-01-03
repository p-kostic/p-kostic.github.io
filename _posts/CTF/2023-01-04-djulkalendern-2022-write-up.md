---
title: dJulkalendern 2022 Write-up
date: 2023-01-04 20:12:30 +0100
categories: [CTF]
tags: [ctf]     # TAG names should always be lowercase
math: true
---

## Introduction

A Swedish friend and student at KTH university introduced me to a CTF-like puzzle in an advent calendar format called [dJulkalendern](https://djul.datasektionen.se/). A window (i.e. puzzle) will be released every weekday at 12:15 PM (Europe/Stockholm, UTC+01:00)

The event is arranged by (I presume students and/or faculty at) KTH University

Luckily, you don't really have to understand Swedish, as the website and puzzles are written in English. The flags are also single English words, without any pre- or postfix.

There also is a Discord server available. Other than some general text and announcement channels, the organizers provided a neat way to grant access to a separate channel, corresponding to each puzzle, if you've successfully solved that challenge. Access is granted through a Discord bot that asks you the "single-english-word-flag" through a DM. 

The content in each Discord channel does force you to use e.g. Google translate to translate some Swedish to your native language if you're interested in what people are saying.

As I work full-time, additionally participating in Advent of Code 2022, and these release at the start of the afternoon, I don't do these exactly when the puzzle releases. Therefore, I won't be ranked high on the scoreboard. 

## Day -1: Ready?

>This window, Window -1, is a practice window of sorts. It does not count toward the final score. The solution is the last word in the notes of the 2003 case, which you can read on the Lore Page.

![Lore](/assets/img/posts/lore.png){: w="667" h="233" }


Navigating to [https://djul.datasektionen.se/lore](https://djul.datasektionen.se/lore) reveals the flag to be `further`.

## Day 1: Mrs. Claus, more like Missing Claus

We are given a password prompt, along with a hint.

```
The root of all 3vil is: 1404928 1481544 1367631 1191016 1030301 970299 1560896 1157625 1367631 1331000
```

The hint is the number `3` in the “l33t speak” version of evil. It shows that a cube root must be performed on these numbers. The fastest way to do this was to open up a web browser's debugger console and write some JavaScript. The resulting numbers are ASCII numbers. JavaScript's `String.fromCharCode` clears that up. The `.join("")` was added for this write up to make it more neat.

```js
"1404928 1481544 1367631 1191016 1030301 970299 1560896 1157625 1367631 1331000"
.split(' ')
.map((x) => String.fromCharCode(Math.cbrt(x))).join("")
```

This gives us the flag `projection`

## Day 2: Writing Safe Code
This day's puzzle instructs you how to use `netcat` to connect to a text-based adventure. This is also known as a Multi User Dungeon or "MUD". 

You traverse the rooms with e.g. `go north`, and find out what there is to interact with in a room by using the `whatishere` command. As you navigate, you collect small clues by e.g. using a stick to shove some things out of reach. The small clues are digits, arranged with underscores in order (e.g. `0 _ _ _`, `_ 3 _ _`, `_ _ 2 _`)

Later, you encounter a safe asking for a pin number.

The last digit was hidden in plain sight, and I didn't get the hint towards it. However, if you have 3 out of 4 digits, you can iterate the safe with a brute-force. The delay of ~20 seconds luckily wasn't increasing on a failed attempt. I luckily wasn't going for time either :)

I thought to myself “I'll start at 1, worst case it's a 0 and it'll take ~200 seconds if I copy the command to my clipboard while waiting”. Of course, the correct digit was the number 9…

![Safe](/assets/img/posts/safe.jpg){: w="428" h="437" }

After opening the safe, the flag was revealed to be `beaned`.

A fun element was that you were allowed to set a username and see who else of the participants was in the room with you through the whoishere command. You were also able to say something in a room to communicate with them. Although I didn't encounter someone else using that feature. Probably all too busy finding the safe pin. 

Later in the Discord channel you hear the hint you missed. Something with 9 chili peppers on a tree, where another place in the game refers to the last digit to be the number of fruits on the hottest plant.

## Day 5: Two Factor Authentication

This day's puzzle gives you a text file, containing a single string of `40501` binary digits. 

`40501` is not divisible by 8 without leaving remainder values. It therefore cannot indicate something in the typical byte length of 8. This is where [CyberChef](https://gchq.github.io/CyberChef/) should come in for the rescue. Can it find something that's not gibberish for other byte lengths? Alas, no clear result. Time to take a step back.

The long character length gravitated me towards a bitmap grayscale image being the solution here. For the possible color values of `0 - 255`, `0` maps to white, and `1` maps to black (or vice-versa). 

To see if this was feasible, we'd need the dimensions of a potential image. We check the factors of the length `40501` through Wolfram Alpha.

![WolframAlpha](/assets/img/posts/wolframalpha.png){: w="798" h="362" }

Exactly two numbers that peak our interest: `101` and `401`! We're on a good track, it doesn't seem that this is just a coincidence.

Before going through the effort of generating an image, maybe it's readable in text form? We assume the text is written horizontally (`401 x 101`). This means that inserting a carriage return after every 401 characters should maybe show something. Hitting `Ctrl + -` a bunch in Visual Studio Code to zoom out, we get the following output. 

![Picturesque](/assets/img/posts/picturesque.png){: w="1161" h="698" }

You can, of course, also write a small program that makes something similar into an actual image, or use online tools like [decode.fr/binary-image](https://www.dcode.fr/binary-image)

## Day 6: Obstacles

For this puzzle, we are given a video where a duck navigates some obstacles. 

<video width="288" height="512" controls style="display: block; margin-left: auto; margin-right: auto;">
  <source src="/assets/img/posts/toktik.mp4" type="video/mp4">
</video>

The clue is in this puzzle's title "Obstacles". If we record the height each set of 3 obstacles have, we get the following numbers.

```
011 120 010 210 111 012 112 202 001 200 221
```

We employ the browser's built-in JavaScript console again for this puzzle. The `parseInt()` function parses a string argument and returns an integer of the specified radix (the base in mathematical numeral systems). In this case, base `3` makes the most sense of course.

```js
[4, 15, 3, 21, 13, 5, 14, 20, 1, 18, 25]
```

We get a set of numbers that seem to be alphabet coded (1 = `a`, 2 = `b`, 3 = `c` etc.). Although `String.fromCharCode()` (and similar functions in most languages) don't have ASCII tables where the `A` character starts at 1, you can of course add the number from where it does: 64 for uppercase or 96 for lowercase.

The final function becomes

```js
"011 120 010 210 111 012 112 202 001 200 221"
.split(' ')
.map((x) => String.fromCharCode(parseInt(x, 3) + 96)).join("")
```

Which returns the flag `documentary`.


## Day 7: Database
This puzzle includes a file called `wishlist.sql` ([download](/assets/files/wishlist.sql)) along with the hint

>check digit 13

This .sql file first contains instructions to create a table called "Books". Each book has an incrementing `id`, `name`, and `isbn` number.

```sql
CREATE TABLE Book (
  id SERIAL PRIMARY KEY,
  name TEXT NOT NULL,
  isbn BIGINT NOT NULL
);
```

Afterwards, there are 250 `INSERT INTO` commands that fills this table with books.

After researching how ISBN-13 numbers work, I quickly figured I'd probably have to check whether they're all valid and go from there.

Not being too eager to set up a new database on my local machine, I employed [sqliteonline.com](https://sqliteonline.com/) to get started much more quickly. I copied and pasted all commands from the file and executed them there in the small sandbox it gives you after connecting.

![PostgreSQL](/assets/img/posts/postgresql.png){: w="668" h="331" }

I then employed the website's export functionality to get a list of all ISBNs easily. I left the tab open to be able to go back anytime. I knew that, with some creative bulk editing functions in Visual Studio Code, I could get this in an easy format such that the numbers could be pasted into a programming language like C#.

Writing the function itself according to the specification seemed like a waste of time. I instead employed Rosetta Code's [ISBN13 check digit](https://rosettacode.org/wiki/ISBN13_check_digit) page to execute the following C# code locally. 

```csharp
var isbns = new List<string>()
{
    "9789432349908", "9780147209354", "9786991874419", "9788913389785", "9789527982560", "9780053807857",
    // Truncated for this write-up.
};

// Source: https://rosettacode.org/wiki/ISBN13_check_digit#C#
static bool CheckISBN13(string code)
{
    code = code.Replace("-", "").Replace(" ", "");
    if (code.Length != 13)
        return false;

    int sum = 0;
    foreach ((int index, char digit) in code.Select((digit, index) => (index, digit)))
    {
        if (char.IsDigit(digit))
            sum += (digit - '0') * (index % 2 == 0 ? 1 : 3);
        else
            return false;
    }
    return sum % 10 == 0;
}

foreach (string isbn in isbns.Where(CheckISBN13))
    Console.WriteLine($"{isbn} is Valid");

```
> Note: Written in .NET 7, thus `static void Main(string[] args) { /* ... */ }` is not required.
{: .prompt-info }


To my surprise only one value was valid: `9789176216132`. 

After checking the corresponding book name with the following query

```sql
SELECT b.name 
  FROM book b 
 WHERE b.isbn = 9789176216132
```

We got the flag `subsequent`.

## Day 8: Only For Developers
The puzzle for the previous day showed the following Note
> Notes: For tomorrows window, you will need to have discord available. You do not necessarily need to join our server, or any server where you can talk to other people. (But you are of course very welcome to join ours :))

Thus this day all participants were given a Discord invite link. After joining the server, a Discord bot called "Locker Lock" would prompt 3 questions in succession. The answers were subsequently  checked after an answer was given to all 3 questions. For example, in order to know if you had question 2 correct, you must've had question 1 correct too. 

> Question 1: What is the street name of the agency?

`Cooking Lane`, found in the [lore](https://djul.datasektionen.se/lore) section of the website.

> Question 2: What is this lock's product number? 

The lock is this very Discord Bot. By copying the Discord ID by right-clicking the bot on the right pane, we get the number by using the "Copy ID" function at the very bottom in the OS's clipboard.

![Discord1](/assets/img/posts/discord1.jpg){: w="204" h="375" }

> Question 3: What is tomtefar's first name?

This one threw me off a short while. A quick google search reveals that “Tomtefar” refers to Father Christmas (Santa Claus) in modern time Swedish. However, "Santa" and other first names given to Santa Claus were not the right answer. However, I then noticed that a Dev called "Tomtefar" was also in this Discord channel. Going to his “About me” link revealed a website with a browser-based [ray tracer](https://magnusson.space/).

![Discord2](/assets/img/posts/discord2.jpg){: w="301" h="646" }

Following the "Source code" link at the top of the page, we are taken to a [GitHub repository](https://github.com/inda21plusplus/mathm-im-already-raytracer)

Although the handle shown for the most recent commit already contained the first name

![Discord3](/assets/img/posts/discord3.jpg){: w="188" h="47" }

we follow to the repository's only contributor's profile: [Mathias Magnusson](https://github.com/mathiasmagnusson)

The first name thus being `Mathias`. 

After having all three questions correct, the bot says the following:

> The locker door opens revealing a dark room full of dusty files. In the very centre of the room there’s a box with “sicily” written in large letters on the front.

and the flag is thus `sicily`

## Day 9: In the lair
This day's puzzle was another MUD that represented a FAT file system as a 5x5 grid of clusters. By traversing the grid using e.g. `go south`, one could use the command `showcontent` to see the contents of a file iff the current cluster is a file. There's one more command in this MUD, `getfat <index>`, which gives us the index of the cell following the one provided. 

The puzzle participants starts in cluster 2

```
00 01 02 03 04
05 06 07 08 09
10 11 12 13 14
15 16 17 18 19
20 21 22 23 24
```

While traversing the grid we encounter a README file in cluster 13 containing the following content

```
HELLO, I AM A VERY EVIL GOOSE.
I HAVE STORED MY CRIMES AND TARGETS IN THE CRIMES FOLDER.
If I have to sound more sophisticated, I store a list of synonyms in the synonyms folder,
which I can liberatingly use when inscribing sentences. 
```


Cluster 7 lists the files of the CRIMES folder. 

| Cluster    | Crime Index | Contents                                                               |
| ---------- |:-----------:| ---------------------------------------------------------------------- |
| Cluster 11 | CRIME A     | https://www.youtube.com/watch?v=dQw4w9WgXcQ (Classic RickRoll URL)     |
| Cluster 8  | CRIME B     | https://www.youtube.com/watch?v=iik25wqIuFo (Alternative RickRoll URL) |
| Cluster 14 | CRIME C     | https://www.youtube.com/watch?v=8ybW48rKBME (Alternative RickRoll URL) |
| Cluster 22 | CRIME D     | Hex code                                                               | 
| Cluster 19 | CRIME E     | https://www.youtube.com/watch?v=xvFZjo5PgG0 (Alternative RickRoll URL) |

`showcontent` in cluster 22 reveals the following content

```
FF D8 FF E0 00 10 4A 46 49 46 00 01 01 01 00 60 00 60 00 00 FF E1 00 68 45 78 69 66 00 00 4D 4D 00 2A 00 00 00 08 00 04 01 1A 00 05 00 00 00 01 00 00 00 3E 01 1B 00 05 00 00 00 01 00 00 00 46 01 28 00 03 00 00 00 01 00 02 00 00 01 31 00 02 00 00 00 11 00 00 00 4E 00 00 00 00 00 01 76 F6 00 00 03 E8 00 01 76 F6 00 00 03 E8 70 61 69 6E 74 2E 6E 65 74 20 34 2E 32 2E 31 34 00 00 FF DB 00 43 00 FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF DB 00 43 01 FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF C0 00 11 08 01 00 01 00 03 01 22 00 02 11 01 03 11 01 FF C4 00 1F 00 00 01 05 01 01 01 01 01 01 00 00 00 00 00 00 00 00 01 02 03 04 05 06 07 08 09 0A 0B FF C4 00 B5 10 00 02 01 03 03 02 04 03 05 05 04 04 00 00 01 7D 01 02 03 00 04 11 05 12 21 31 41 06 13 51 61 07 22 71 14 32 81 91 A1 08 23 42 B1 C1 15 52 D1 F0 24 33 62 72 82 09 0A 16 17 18 19 1A 25 26 27 28 29 2A 34 35 36 37 38 39 3A 43 44 45 46 47 48 49 4A 53 54 55 56 57 58 59 5A 63 64 65 66 67 68 69 6A 73 74 75 76 77 78 79 7A 83 84 85 86 87 88 89 8A 92 93 94 95 96 97 98 99 9A A2 A3 A4 A5 A6 A7 A8 A9 AA B2 B3 B4 B5 B6 B7 B8 B9 BA C2 C3 C4 C5 C6 C7 C8 C9 CA D2 D3 D4 D5 D6 D7 D8 D9 DA E1 E2 E3 E4 E5 E6 E7 E8 E9 EA F1 F2 F3 F4 F5 F6 F7 F8 F9 FA FF C4 00 1F 01 00 03 01 01 01 01 01 01
```

When using [CyberChef](https://gchq.github.io/CyberChef/)'s From Hex recipe on the hex code in Cluster 22, we encounter the following

```
ÿØÿà..JFIF.....`.`..ÿá.hExif..MM.*.................>...........F.(...........1.........N......vö...è..vö...èpaint.net 4.2.14..ÿÛ.C.ÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÛ.C.ÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÀ.........".......ÿÄ............................	
.ÿÄ.µ................}........!1A..Qa."q.2..¡.#B±Á.RÑð$3br.	
.....%&'()*456789:CDEFGHIJSTUVWXYZcdefghijstuvwxyz.................¢£¤¥¦§¨©ª²³´µ¶·¸¹ºÂÃÄÅÆÇÈÉÊÒÓÔÕÖ×ØÙÚáâãäåæçèéêñòóôõö÷øùúÿÄ...........
```

Notably **JFIF**, but also the EXIF information **paint.net**, indicates that this hex code is part of an JPEG-image that needs to be reconstructed.

By using the `getfat 22` command, we get `0x9` (i.e. integer `9`)

The contents of cluster 9, using the `showcontent` command, are

```
01 01 01 00 00 00 00 00 00 01 02 03 04 05 06 07 08 09 0A 0B FF C4 00 B5 11 00 02 01 02 04 04 03 04 07 05 04 04 00 01 02 77 00 01 02 03 11 04 05 21 31 06 12 41 51 07 61 71 13 22 32 81 08 14 42 91 A1 B1 C1 09 23 33 52 F0 15 62 72 D1 0A 16 24 34 E1 25 F1 17 18 19 1A 26 27 28 29 2A 35 36 37 38 39 3A 43 44 45 46 47 48 49 4A 53 54 55 56 57 58 59 5A 63 64 65 66 67 68 69 6A 73 74 75 76 77 78 79 7A 82 83 84 85 86 87 88 89 8A 92 93 94 95 96 97 98 99 9A A2 A3 A4 A5 A6 A7 A8 A9 AA B2 B3 B4 B5 B6 B7 B8 B9 BA C2 C3 C4 C5 C6 C7 C8 C9 CA D2 D3 D4 D5 D6 D7 D8 D9 DA E2 E3 E4 E5 E6 E7 E8 E9 EA F2 F3 F4 F5 F6 F7 F8 F9 FA FF DA 00 0C 03 01 00 02 11 03 11 00 3F 00 65 14 51 40 0B 4A 29 B4 B9 A0 09 29 A6 93 34 84 D2 18 94 52 8E B4 EF D7 8A 62 1B 4B 83 F9 52 E3 9C 7E 22 82 68 01 B4 51 45 00 1D E8 A5 14 94 00 52 52 D2 50 01 45 18 A5 A0 04 A2 96 8E 7B 0A 00 51 48 69 E1 69 0A F1 40 0C A2 8A 30 68 00 A2 8A 5C 1A 00 29 28 A2 80 0A 28 A5 C5 00 25 48 31 4C 34 66 80 0C 51 8A 71 C5 36 81 B1 28 A7 51 40 86 D1 45 39 71 DE 80 12 94 74 EB 4E C7 3C 7E 54 70 28 00 FE 78 A4 C1 C7 4A 75 27 4A 00 4C 7F 9F 5A 69 EB 4E CD 34 FA D0 02 D2 52 51 40 05 14 52 E2 80 B0 94 B8 A5 A4 A0 AE 5E E1 52 01 C5 47 52 0E 82 80 63 A9 29 69 28 24 88 8C 1A 5F FF 00 5F FF 00 5A 86 EB 47 E3 9A 00 5C 0E BF 43 47 D7 D3 14 60 F6 E2 90 FD 73 40 0D A2 8C 1A 28 01 C0 53 F0 29 80 D2 E6 90 C4 34 DA 53 CD 14 C4 28 A3 1F CE 93 A5 1C D0 36 83 F5 A2 90 D1 9A 04 2D 1D 0D 14 A3 1D E8 01 FF 00 4A 4C E3 3E BD E8 DC 29 01 EB EF 40 0E A4 26 8F 6E 98 A4 3D 70 47 1E B4 00 D3 45 06 8A 00 4A 5C 0A
```

Using the `getfat 9` command, we get `0x11` (i.e. integer `17`)

The contents of cluster 17, using the `showcontent` command, are

```
05 14 14 90 B4 52 51 48 61 45 14 50 01 40 38 A2 92 98 99 20 6A 09 ED FA D3 3B D2 8E FF 00 D6 82 45 FE 82 8C 51 9C 0C 9E F4 B4 00 98 CD 03 BD 2D 1C 1C D0 02 74 3F CA 90 9C 0A 5F AE 73 ED 48 47 F9 34 00 DA 28 A5 A0 04 A2 8A 28 01 E0 73 CF 41 4E EB CF 6A 69 39 A5 07 02 90 03 0E 2A 3A 90 9C D3 08 A0 62 51 45 2E 29 85 84 A7 0A 4A 33 40 31 E7 A5 19 F5 FF 00 F5 53 32 68 E4 D0 20 34 94 52 81 9A 00 05 2D 3B 69 A3 6D 05 26 AC 32 8A 93 68 A5 C0 F4 A0 2E 88 E8 C1 A9 31 4B 40 73 11 6D 34 DA 9A A2 23 93 41 2D DC 4A 5A 4A 77 6F 7A 00 5E 31 FC FD 68 07 34 99 C5 28 3F AE 68 00 CE 3D E9 0F 1D B8 34 B9 E2 9A 68 00 CD 04 E6 92 96 80 12 8A 76 28 C5 00 36 8A 28 A0 07 51 49 45 00 14 EA 6F FF 00 AA 9C 29 02 76 0C 51 4B EF 47 4A 0A B8 DA 0F 4A 5C 52 50 3D D0 01 4F A6 2D 3B 34 12 04 50 9D E9 09 A7 2F 4A 10 87 51 45 14 C0 28 A2 8A 00 28 A2 93 34 00 B5 13 75 A9 33 4C 34 00 94 99 A2 8A 00 3A D1 4B 45 00 25 14 51 40 09 4A 28 A4 A0 09 29 0D 37 34 66 90 C3 14 52 D2 53 10 1A 28 CD 14 00 53 81 A6 D3 87 F2 A0 07 75 FC 29 33 40 E6 9D 8A 40 37 AF E1 4B 47 4E 94 DA 00 6F AD 19 A3 BD 25 30 0A 78 38 E0 D3 05 38 D0 03 F7 0F 5A 33 51 AF 5A 92 93 76 18 13 81 40 39 E2 82 32 31 40 18 A5 7D 04 2D 21 E9 4B 45 21 8C 53 D4 52 D0 00 14 51 D4 10 CE 86 8C D0 DD 69 2A C4 14 51 4B 40 09 4B 4E 00 1A 76 D1 40 11 8A 71 00 53 F6 8A 4C 0A 00 8C D2 53 A9 B4 00 51 47 BD 14 00 52 E2 8A 78 39 ED 40 DA B0 C0 0F 6A 78 1C 53 B1 45 02 12 8C FB 52 D1 40 0C 26 93 34 E2 29 A3 8F CE 90 09 DE 8A 28 A6 02 52 E6 92 8A 00 55 EB 52 54 43 AD 48 2A 58 FA 0E A2 93 38 A3 70 A4 02 D1 4D DC
```

Using the `getfat 17` command, we get `0x5` (i.e. integer `5`)

The contents of cluster 5, using the `showcontent` command, are

```
28 DC 3D 28 B0 59 8E A4 34 DD D9 E9 45 16 18 8D 4D A5 34 55 09 A1 29 71 49 45 31 12 2D 3A 9A BD 29 F4 00 52 52 D2 50 04 46 92 94 F5 34 01 9A 00 5C 0C 75 14 D0 2A 4D B4 D2 29 00 95 20 18 A8 C7 51 52 D3 1B 16 8A 28 A0 41 45 14 50 02 53 1B D6 9F 4D 23 34 00 CC 52 54 D8 A6 95 A0 06 62 96 97 06 92 91 6A C1 4A 29 28 A0 63 E9 36 D0 0D 3A A4 9D 86 6D A3 6D 49 49 4F 51 5C 66 28 34 A4 D3 0F 34 14 14 51 4B 4C 02 90 D2 D1 40 35 70 04 8A 76 E3 4C A5 A0 2C 3B 71 A4 C9 A4 A2 80 B2 1B DE 9E B4 CA 70 34 C9 25 A6 9A 4C D2 13 48 43 47 15 25 47 4F 07 34 C0 75 2D 36 96 80 16 92 8A 4A 00 5A 41 49 9A 75 00 2D 14 51 40 05 30 8A 7D 30 D0 34 36 8A 28 A4 58 51 45 14 00 B9 3E B4 99 3E B4 51 40 AC 25 2D 14 50 30 A2 8A 28 00 A2 8A 28 10 94 B4 94 50 02 D3 4D 2D 25 31 36 14 51 45 04 86 68 A2 8A 00 28 E9 45 14 00 F0 69 73 51 D1 93 40 12 66 93 34 DC 9A 4C 9A 06 38 D3 94 FA D2 0E 94 50 21 F9 1E B4 64 7A D3 28 A0 07 12 29 9C D3 87 26 9F 40 D3 B1 17 34 62 A5 A2 80 E6 22 C1 A3 06 A4 A5 A0 39 88 B0 68 C1 A9 68 A0 39 88 B0 7D 0D 18 3E 95 2D 14 07 31 16 0F A5 18 3E 95 25 2D 01 76 43 45 3D FB 53 28 1D C2 92 8A 3E B4 05 C2 8A 53 D7 8A 4A 09 0A 28 A2 80 12 96 8A 28 00 A2 8A 78 5C 8E B4 00 CA 29 FB 68 C5 03 B2 19 46 29 FB 68 DB 40 F4 13 B7 E1 4C A7 F6 A6 50 48 51 4A 06 4E 05 3B 61 F6 FF 00 3F 85 00 32 8A 30 68 A0 02 8A 28 A0 02 8A 28 A0 02 8A 28 A0 02 8A 28 A0 02 9C 9F 78 53 69 C9 F7 85 00 3D FA 0A 8E A4 7E 82 A3 A0 68 29 29 68 A0 41 45 14 50 01 45 2D 14 00 94 52 D1 40 05 3D 7A 0A 8E 9E 0F D2 80 1F 45 37 3F 4F D6 8C 9F F3 9A 00 5A 29 B9 A3 26 81 89 D8 FE
```

Using the `getfat 5` command, we get `0x3` (i.e. integer `3`)

The contents of cluster 3, using the `showcontent` command, are

```
34 CA 7F 6A 65 02 0A 94 0F 97 EB FD 7F FA D5 15 3D 9B 3C 0E 9D E8 01 FF 00 C3 81 DF FC E6 9A C0 05 F7 F5 A0 B0 38 03 A7 7F F0 A4 62 A7 B9 E3 B6 28 01 02 93 FF 00 D7 A4 2A 47 5A 79 65 38 EB C1 E9 48 4A 93 9C 9F CA 80 13 69 C0 3E B4 84 11 8C F7 A7 EF 19 1E 94 84 82 41 E7 8E D8 A0 04 DA 69 08 20 E3 BD 4B 8C 90 7D 07 4F AD 37 23 76 79 34 00 DD 87 DB F3 A3 69 F6 FC E9 DB 94 12 79 24 D4 74 00 53 93 EF 0F F3 DA 9B 4E 4F BC 28 01 EF D0 54 75 23 F6 A8 E8 00 A2 8A 28 00 A5 A4 A5 A0 02 8A 4A 5A 00 28 A2 8A 00 29 45 25 14 00 EA 33 4D A5 A0 03 34 99 A2 8A 06 1D A9 B4 F0 32 38 A4 DA 7D 28 10 DA 29 DB 5B D2 8D AD E9 40 0D A2 9D B5 BD 28 DA 7D 28 01 B4 53 B6 B7 A5 1B 5B D2 80 1B 45 3B 6B 7A 51 B5 BD 28 00 2E 71 FC CD 36 9D B4 D1 B4 D0 03 68 A7 6D 34 6D 3F E4 D0 03 69 C9 F7 87 F9 ED 46 C3 FE 4D 39 54 82 09 A0 05 7E D5 1D 3D C8 A6 50 01 4B 49 45 03 16 8A 4A 5A 00 28 A2 8A 04 14 51 45 00 14 51 4B 40 C2 8A 28 A0 03 14 51 40 A0 41 8A 4A 71 3C 53 28 00 A2 8A 28 00 A2 8A 5C 50 02 52 D1 45 00 14 51 F8 51 40 05 25 2D 14 00 51 49 45 00 2D 28 A6 D3 85 00 04 0E D4 94 A6 8A 00 4A 29 68 A0 62 51 4B 45 00 25 14 51 40 05 14 51 40 05 14 51 40 82 8A 28 A0 02 8A 4A 28 01 FD 69 B4 51 40 05 1F 8D 25 14 00 B4 71 49 45 00 2F 1E F4 71 49 45 00 2D 14 94 50 02 D2 51 45 00 14 51 45 00 2E 07 AD 02 92 96 80 0A 51 49 40 A0 05 A2 8A 28 18 52 52 D2 50 01 45 14 50 01 45 14 50 20 A2 8A 28 01 70 4D 2E D3 E9 49 C8 E8 68 CB 7A D0 02 ED 3E 94 60 D2 64 FA D1 93 EB 40 0B 83 49 B4 D2 64 FA D2 E4 D0 02 ED 3E 82 93 07 DA 8C 9F 5A 32 7D 68 00 C1 F4 14 B8 3E 82 93 27 D6
```

Using the `getfat 3` command, we get `0x4` (i.e. integer `4`)

The contents of cluster 4, using the `showcontent` command, are

```
8C 9F 5A 00 5C 1F 41 46 0F A0 A4 C9 F5 A4 C9 F5 A0 07 60 FA 0A 36 9A 6F 3E B4 73 EB 40 0B B4 D1 B4 D2 73 EB 47 3E B4 00 BB 4D 1B 4D 27 3E B4 73 EB 40 0B B4 D2 ED 34 DE 7D 69 79 F5 A0 00 82 29 05 1C FA D1 40 0B 45 14 94 0C 28 A2 8A 04 14 51 45 00 14 51 45 00 7F FF D9
```

Although, by length, this clearly marks the end of the file. Using the `getfat 4` command returns `0xFFF` (i.e. integer `4095`).

Using [CyberChef](https://gchq.github.io/CyberChef/)'s "Render Image" recipe with the Input format being `Hex`, we get the following output when copying and pasting the hex code in the order shown above.

![CyberImage](/assets/img/posts/cyberimage.PNG){: w="792" h="563" }

Revealing the flag `mango`

## Day 12: With Love from Julius

This day's puzzle contains a [rebus](https://en.wikipedia.org/wiki/Rebus)-like image. The flag is computed from solving this rebus.

![Cipher](/assets/img/posts/cipher.png){: w="2724" h="1294" }

We start at the top and go from left to right. The first segment being

$$ ((110\ 0011 + (-1)^{P(heads)} + x) \leq 50 $$

* The binary string $110\ 0011$ refers to the `s` character
* $P(heads)$ refers to the chance of heads when flipping a coin, equivalent to `0.5`. Evaluating $(-1)^{0.5}$ (i.e. $\sqrt{-1}$) gives us the [imaginary unit](https://en.wikipedia.org/wiki/Imaginary_unit) $i$.

$$ ((s + i + x) \leq 50 $$

`6` is indeed smaller than `50`, giving us the value `true` for this expression.

Beneath is a hangman guessing game that's in progress. Using a [Hangman Solver](https://www.hanginghyena.com/hangmansolver) reveals the only candidate word to be `zero`.

Both of these are fed into what seems to be a MIL/ANSI Symbol for an [OR gate](https://simple.wikipedia.org/wiki/OR_gate). Only the top expression was true, so we pass the value `true` into the  [syntactic sugar](https://en.wikipedia.org/wiki/Syntactic_suga) for a Python string slice (i.e. substring). 

```python
>>> "true"[:2]
'tr'
```

We've completed the first top half of the rebus; Now for the bottom half.

We first encounter a heart shape added to a drawing of a city within parentheses. Outside the parentheses is a triangle. I considered the triangle to be the upper-case Greek symbol for delta ($\Delta$).

This is then divided by a drawing of a clock.

After a while, I figured that the two arrows coming from the heart shape indicate a shuffle of two sets of characters within the word `love`, as this would be the only thing that would make sense when concatenating it with the word `city`. `love` would thus become `velo` + `city` = `velocity`.

The bottom half represented the word `time`, giving us the mathematical formula

$$\frac{\Delta velocity}{\Delta time}$$

Which is how `acceleration` is calculated in physics.

Next, this word is fed into what looks to be a rotating disc with 26 segments and a coin depicting Julius Caesar. This refers to a [Caesar Cipher](https://en.wikipedia.org/wiki/Caesar_cipher) (ROT6). The title of this day's window also gave a hint towards this. Using the `ROT13` recipe in [CyberChef](https://gchq.github.io/CyberChef/) with an Amount of 6, we get the output `giikrkxgzout`.

Evaluating the Python string slice in a Python interpreter

```python
>>> "giikrkxgzout"[9:]
'out'
```

Resulting in the flag `tr` + `out` = `trout`


## Day 13: One Way

The puzzle of Day 13 linked to a single HTML page

```html
<!DOCTYPE html>
<html>
<head>
    <title>Definitely a puzzle in a newspaper</title>
</head>
<body>
    <input placeholder="password" type="text">
    <p></p>
    <script src="index.js"></script>
    <script>
        const p = document.querySelector("p");
        const input = document.querySelector("input");
        input.addEventListener("keyup", function() {
            const valid = verifyPassword(this.value);
            p.style = `color: ${valid ? "green" : "red"}`;
            p.textContent = valid ? "Congrats!" : "Wrong password :(";
        });
    </script>
</body>
</html>
```

The script source `index.js` contained 103 lines of the following code. I've truncated a part of it for the purpose of this write-up. The puzzle seems to be some sort of _security through obscurity_ function nesting, containing boolean logic, that you have to reduce.

```js
// Truncated to 16 lines from 103
const zmp = s => false;
const by4 = s => s[27] == 'u' && dly(s);
const hgj = s => s[32] == 't' && efh(s) || s[10] == 'R' && sfa(s) || s[22] == 's' && t3v(s);
const kke = s => false;
const lf1 = s => false;
const az4 = s => s[21] == 'y' && js1(s);
const wjz = s => s[40] == '4' && n15(s) || s[37] == 't' && nxm(s) || s[20] == 't' && yj6(s) || s[26] == 'o' && e31(s);
const vxu = s => s[24] == 'w' && opc(s) || s[40] == 't' && rvp(s);
const yq8 = s => s[7] == 'C' && n15(s);
const tdu = s => s[7] == 'X' && e31(s) || s[9] == 'x' && n15(s) || s[23] == '9' && ybd(s) || s[10] == 'l' && yj6(s);
const sfa = s => s[24] == 'o' && e31(s) || s[40] == 'J' && glo(s);
const i2g = s => s[33] == ' ' && ovt(s) || s[14] == 'h' && hum(s) || s[31] == '3' && oxa(s) || s[8] == 'z' && tdu(s);
const hum = s => false;
const w1j = s => s[31] == 'S' && nxm(s) || s[14] == '7' && ybd(s) || s[18] == 'y' && kf9(s) || s[35] == 'Y' && lf1(s);
const bpc = s => s[8] == 'j' && n15(s) || s[29] == 'L' && yj6(s);
const verifyPassword = a70;
```

The above snippet shows that certain 3 character `const` functions with the parameter `s` that immediately evaluate to `false`. Using Visual Studio Code's _Find and Replace_ functionality, we can reduce a lot of the code that verifies the password. Afterwards, we sort each row in order of the zero-based index number of the string parameter `s`.

```js
const verifyPassword = a70;
const cus = s => s[0]  == 'T' && oai(s);
const hx2 = s => s[1]  == 'h' && n75(s);
const puf = s => s[2]  == 'e' && s6w(s);
const opc = s => s[3]  == ' ' && tla(s);
const a70 = s => s[4]  == 'p' && ef0(s);
const m8z = s => s[5]  == 'a' && puf(s);
const rsi = s => s[6]  == 's' && p17(s);
const etz = s => s[7]  == 's' && rsi(s);
const nag = s => s[8]  == 'w' && q24(s);
const bhw = s => s[9]  == 'o' && xu1(s);
const t6x = s => s[10] == 'r' && i2g(s);
const j1o = s => s[11] == 'd' && epw(s);
const n75 = s => s[12] == ' ' && f4j(s);
const yy9 = s => s[13] == 'f' && vxu(s);
const y65 = s => s[14] == 'o' && hgj(s);
const xu1 = s => s[15] == 'r' && v3a(s);
const s35 = s => s[16] == ' ' && nag(s);
const smg = s => s[17] == 't' && q5m(s);
const s3u = s => s[18] == 'o' && yy9(s);
const oai = s => s[19] == 'd' && t6x(s);
const atb = s => s[20] == 'a' && bhw(s);
const az4 = s => s[21] == 'y' && js1(s);
const hgj = s => s[22] == 's' && t3v(s);
const ef0 = s => s[23] == ' ' && smg(s);
const vxu = s => s[24] == 'w' && opc(s);
const tla = s => s[25] == 'i' && cqx(s);
const txk = s => s[26] == 'n' && atb(s);
const t3v = s => s[27] == 'd' && vz0(s);
const ovt = s => s[28] == 'o' && txk(s);
const js1 = s => s[29] == 'w' && etz(s);
const cqx = s => s[30] == ' ' && hx2(s);
const v3a = s => s[31] == 'i' && ea6(s);
const bz6 = s => s[32] == 's' && az4(s);
const i2g = s => s[33] == ' ' && ovt(s);
const p17 = s => s[34] == 't' && j1o(s);
const s6w = s => s[35] == 'h' && y65(s);
const q24 = s => s[36] == 'r' && s3u(s);
const vz0 = s => s[37] == 'o' && cus(s);
const epw = s => s[38] == 'u' && s35(s);
const f4j = s => s[39] == 'g' && m8z(s);
const q5m = s => s[40] == 'h' && bz6(s);
const ea6 = s => s[41] == undefined;
```

Reading each character from top to bottom, we get `The password for todays window is through`. The flag thus being `through`.


## Day 14: The plot thickens

This day's puzzle only contained to a small 16x11 image, called `cipher.png`

![Cipher2](/assets/img/posts/cipher2.png){: w="16" h="11" }

The title _"The **plot** thickens"_ and the text in the puzzle's story

>You had a gut feeling that something was off. The whole situation smelled of feathers. You could sense that there was a **plot** somewhere, but who or what it was, was still shrouded in mystery.

indicate that the values need to be _plotted_. We are, however, working with a PNG image that also contain alpha values. What do we exactly plot?

Loading [CyberChef](https://gchq.github.io/CyberChef/), uploading the image as the input, and using the _Extract RGBA_ recipe, I noticed that every 4th value (the alpha value) was `255` anyway. It seemed that the RGB values themselves would need to represent the 3D $(x,y,z)$ coordinates.

After some trial-and-error Google queries looking for a way to plot this quickly in e.g. _R_ or Python's _matplotlib_, I encountered the website [franciscouzo.github.io/image_colors](https://franciscouzo.github.io/image_colors/) that extracts the RGB values as sets of coordinates, and immediately plots them in 3D for you. After rotating the camera, we encounter the following shape.

![Pikachu](/assets/img/posts/pikachu.png){: w="503" h="492" }

The flag thus being `pikachu`.

## Day 15: Goose Hunting

Today's puzzle includes a file called `cipher.txt`. The puzzle text contained a reference to Sparta. By querying Google for _"ciphers in spartan times"_ it's revealed we're dealing with a [Scytale Cipher](https://en.wikipedia.org/wiki/Scytale).

> .. Before that she was known as Helen of **Sparta**, but that’s a **tale*** as old as the **sky**.” You approach a large gate, leading into a gated community with even larger houses than those you’ve already passed. On the locked gate there is a keyboard glued onto the gate, next to a screen which asks you for the **‘longest word’**. Alongside it is a long band of seemingly random letters.

I've also found that this cipher is functionally similar to a [Caesar Box cipher](https://www.cachesleuth.com/caesarbox.html)

We employ CyberChef again, as it has Caesar Box Cipher functionality. A trial-and-error achieved Box Height of `28` as a parameter gives us the following text.

> Dear fellow Spartan,  As I sit here, armor shining and spear at the ready, I can't help but reflect on the journey that has brought us to this point. For as long as I can remember, being a Spartan warrior has been my ultimate goal.  As a child, I was taken from my family and brought to the Agoge, where I began my training. It was tough, but I was determined to become the best warrior I could be. I spent countless hours practicing with my weapons, honing my skills and building my strength.  But the training wasn't just about physical prowess. We were also taught the values that make a true Spartan: courage, discipline, and loyalty. These were the principles that would guide us in battle, and in all aspects of our lives.  As I grew older and moved up the ranks, I took part in many battles, defending our city from those who would seek to do it harm. And now, here we are, preparing to face our greatest challenge yet.  But I am not afraid. I know that each and every one of us is ready for this moment. We have dedicated our lives to this cause, and we will not falter.  As we prepare for battle, I wanted to share with you our plan for victory. We will strike at dawn, when our enemies are at their most vulnerable.  Our strategy is simple: we will divide into two groups, one attacking from the front and the other from the rear. This will catch our enemies off guard, and give us the upper hand.  Once we have engaged the enemy, we will fight with all the skill and strength that our training has given us. We will not falter, we will not retreat. We will stand our ground and emerge victorious.  But above all else, we must remember our discipline. We must not be swayed by anger or fear, but focus on the task at hand. If we stay true to our training and our values, victory will be ours.  So let us go forth with courage and **determination**. We are Spartans, the finest warriors in all the land. Let us go forth, brothers in arms, and face our enemies with courage and **determination**. We will show them the strength of the Spartan spirit, and we will emerge victorious. Today is the day, we prove it!  Yours in battle, A fellow Spartan soldier.

![Recipe](/assets/img/posts/recipe.png){: w="576" h="492" }


Using the CyberChef recipe above, it's revealed that the longest word and thus the flag is `determination`.


## Day 16: Too many mirrors

This day's puzzle was another MUD. Similar to the previous MUDs, we use e.g. `go east` to navigate the map. Certain rooms in the map contained mirrors, aimed horizontally or vertically, that transforms the whole map according to the mirror's orientation. We `interact` with the mirror to flip the map. Certain rooms also only let the player move in a certain direction (e.g. only east or west). These constraints remain put when the map is flipped by a mirror, letting the player proceed by interacting with them.

Once you've reached the exit in the room, you'll be put into the next room.

If you get stuck, you can always reset the floor by using the `restartfloor` command.

### Floor 1

![Floor1Map](/assets/img/posts/floor1.png){: w="566" h="368" }

The starting position contains a vertical mirror ($V_m$) to the east. By flipping the map vertically here, we can go _east_ twice to counteract this constraint.

Similar for the southern _only N_ room, we first flip the map horizontally by going to the northern mirror ($H_m$). 

The same idea applies for the other constraint rooms and their respective mirrors. In order to _solve_ floor 1, we are required to `interact` with every mirror except for the $V_m$ to the south-east. 

### Floor 2

This floor added an extra dynamic to the puzzle: a _shiny orb_ that you can place in a single room such that you can ignore that room's movement limitations. However, when flipping the map, the orb stays put.

![Floor2Map](/assets/img/posts/floor2.png){: w="523" h="205" }

Rooms 3, 7, and 13 force you to use the $V_m$ to the west first in order to be able to reach the horizontal mirror. Afterwards the orb must me placed in room 7 as it's relative to room 5. Then, after interacting with the horizontal mirror in room 14 to be able to go past room 9, we go to towards the exit. To "make the orb go" from room 7 to room 5 we flip the vertical mirror in room 11 along the way towards the exit.

### Floor 3

> You open a door and find yourself in a relatively well-lit but small space with a ceiling made out of wood and the ground being mostly covered by sand. On the wall in front of you there are two signs. One pointing left saying A and HOTEL AURORE and one pointing right saying B and PALAIS DE LA KASBAH. To your left there is what seems to be a driveway acending upwards.

* To the west: Mid CT
* To the east: A Ramp

This immediately reminded me of the map `de_dust2` from the online first-person shooter Counter-Strike. This map contains a corner called __Goose__ close to __A Ramp__ and __A site__. If we `go east` towards `A Ramp`, we follow-up with `go north` to go towards `Goose`.

>YOU HAVE FOUND THE EVIL GOOSE. here is the password: cultural

The flag thus being `cultural`.

## Day 19: In the interrogation room

This day's puzzle contained a link with a file called `decompiler.dll`, with the puzzle's text being

>“As I suspected. Quite a few files that would have run, were it not for my expert knowledge of bash… Let’s see what we got here. dAnkan says while opening a compiled C#-file, getting ready to decrypt it.

The first thing I tried was using [JetBrains dotPeek](https://www.jetbrains.com/decompiler/). However, the following was returned.

```csharp
// Decompiled with JetBrains decompiler
// Type: Program
// Assembly: decompiler, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null
// MVID: 4E4DE787-C7B3-4A7D-E49B331E62C0
// Assembly location: C:\Users\Petar Kostic\Downloads\decompiler.dll

using System;

Console.WriteLine("the decompiler is too smart! :((");
```

Fortunately, right clicking the output in dotPeek allows us to see Microsoft's Common Intermediate Language (CIL) 

```bpf
// Type: Program 
// Assembly: decomplier, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null
// MVID: 4E4DE787-C7B3-4A7D-A7DC-E49B331E62C0
// Location: C:\Users\Petar Kostic\Downloads\decomplier.dll
// Sequence point data from decompiler

.class private auto ansi beforefieldinit
  Program
    extends [System.Runtime]System.Object
{
  .custom instance void [System.Runtime]System.Runtime.CompilerServices.CompilerGeneratedAttribute::.ctor()
    = (01 00 00 00 )

  .method private hidebysig static void
    '<Main>$'(
      string[] args
    ) cil managed
  {
    .entrypoint
    .maxstack 8

    IL_0000: ldstr        ".entrypoint\r\n    // Code size       98 (0x62)\r\n    .maxstack  3\r\n    .locals init (int32[] V_0,\r\n             int32 V_1,\r\n             int32[] V_2,\r\n             int32 V_3,\r\n             int32 V_4)\r\n    IL_0000:  ldc.i4.s   10\r\n    IL_0002:  newarr     [System.Runtime]System.Int32\r\n    IL_0007:  stloc.0\r\n    IL_0008:  ldloc.0\r\n    IL_0009:  ldc.i4.0\r\n    IL_000a:  ldc.i4.0\r\n    IL_000b:  stelem.i4\r\n    IL_000c:  ldloc.0\r\n    IL_000d:  ldc.i4.1\r\n    IL_000e:  ldc.i4.2\r\n    IL_000f:  stelem.i4\r\n    IL_0010:  ldloc.0\r\n    IL_0011:  ldc.i4.2\r\n    IL_0012:  ldc.i4.s   -3\r\n    IL_0014:  stelem.i4\r\n    IL_0015:  ldloc.0\r\n    IL_0016:  ldc.i4.3\r\n    IL_0017:  ldc.i4.s   -11\r\n    IL_0019:  stelem.i4\r\n    IL_001a:  ldloc.0\r\n    IL_001b:  ldc.i4.4\r\n    IL_001c:  ldc.i4.s   17\r\n    IL_001e:  stelem.i4\r\n    IL_001f:  ldloc.0\r\n    IL_0020:  ldc.i4.5\r\n    IL_0021:  ldc.i4.s   -18\r\n    IL_0023:  stelem.i4\r\n    IL_0024:  ldloc.0\r\n    IL_0025:  ldc.i4.6\r\n    IL_0026:  ldc.i4.s   17\r\n    IL_0028:  stelem.i4\r\n    IL_0029:  ldloc.0\r\n    IL_002a:  ldc.i4.7\r\n    IL_002b:  ldc.i4.s   -11\r\n    IL_002d:  stelem.i4\r\n    IL_002e:  ldloc.0\r\n    IL_002f:  ldc.i4.8\r\n    IL_0030:  ldc.i4.s   13\r\n    IL_0032:  stelem.i4\r\n    IL_0033:  ldloc.0\r\n    IL_0034:  ldc.i4.s   9\r\n    IL_0036:  ldc.i4.s   -17\r\n    IL_0038:  stelem.i4\r\n    IL_0039:  ldc.i4.s   112\r\n    IL_003b:  stloc.1\r\n    IL_003c:  nop\r\n    IL_003d:  ldloc.0\r\n    IL_003e:  stloc.2\r\n    IL_003f:  ldc.i4.0\r\n    IL_0040:  stloc.3\r\n    IL_0041:  br.s       IL_005b\r\n\r\n    IL_0043:  ldloc.2\r\n    IL_0044:  ldloc.3\r\n    IL_0045:  ldelem.i4\r\n    IL_0046:  stloc.s    V_4\r\n    IL_0048:  nop\r\n    IL_0049:  ldloc.1\r\n    IL_004a:  ldloc.s    V_4\r\n    IL_004c:  add\r\n    IL_004d:  stloc.1\r\n    IL_004e:  ldloc.1\r\n    IL_004f:  conv.u2\r\n    IL_0050:  call       void [System.Console]System.Console::Write(char)\r\n    IL_0055:  nop\r\n    IL_0056:  nop\r\n    IL_0057:  ldloc.3\r\n    IL_0058:  ldc.i4.1\r\n    IL_0059:  add\r\n    IL_005a:  stloc.3\r\n    IL_005b:  ldloc.3\r\n    IL_005c:  ldloc.2\r\n    IL_005d:  ldlen\r\n    IL_005e:  conv.i4\r\n    IL_005f:  blt.s      IL_0043\r\n\r\n    IL_0061:  ret"
    IL_0005: pop

    // [9 1 - 9 54]
    IL_0006: ldstr        "the decompiler is too smart! :(("
    IL_000b: call         void [System.Console]System.Console::WriteLine(string)
    IL_0010: ret

  } // end of method Program::'<Main>$'

  .method public hidebysig specialname rtspecialname instance void
    .ctor() cil managed
  {
    .maxstack 8

    IL_0000: ldarg.0      // this
    IL_0001: call         instance void [System.Runtime]System.Object::.ctor()
    IL_0006: ret

  } // end of method Program::.ctor
} // end of class Program
```

The long string on line 22 that starts with `IL_0000: ldstr`. Here a string is loaded with a set of instructions. We substitute the lines within the range [`19 - 28`] (i.e. `.entrypoint` through `IL_0010: ret`) with the instructions from the string.

We paste this into [SharpLab.io](https://sharplab.io/), converting from `IL` to `C#`, to get the following output.

```csharp
using System;
using System.Runtime.CompilerServices;   

[CompilerGenerated]
internal class Program
{
    public static void Main(string[] args)
    {
        int[] array = new int[10];
        array[0] = 0;
        array[1] = 2;
        array[2] = -3;
        array[3] = -11;
        array[4] = 17;
        array[5] = -18;
        array[6] = 17;
        array[7] = -11;
        array[8] = 13;
        array[9] = -17;
        int num = 112;
        int[] array2 = array;
        int num2 = 0;
        while (num2 < array2.Length)
        {
            int num3 = array2[num2];
            num += num3;
            Console.Write((char)num);
            num2++;
        }
    }
}
```

Running this `.cs` file through [.NET Fiddle](https://dotnetfiddle.net/) or using `dotnet run`, we get the flag `productive`.

## Day 20: Just another passenger

This day's puzzle shows some instructions, along with an image

```
WoRks evEn foR youR tYpEWriTeR
```

1. Bury The CAPITAL. Take It DOWN, Take It DOWN, Is What's RIGHT.
2. The lower rise up, rise left.

The image shows 5 instances of a 7-segment display where each segment displays a character or a number.

![Display](/assets/img/posts/display.png){: w="174" h="51" }

* For the first instruction
    1. We first get a the set of uppercase characters (i.e. capitals) from the instructions: `WRERRYEWTR`
    2. We perform a [Keyboard Shift Cipher](https://www.dcode.fr/keyboard-shift-cipher) of Down -> Down -> Right on the characters: `CBVBBMVCNB`

* For the second instruction
    1. We get the set of lowercase characters `oksevnfoyoutprie`
    2. We perform a [Keyboard Shift Cipher](https://www.dcode.fr/keyboard-shift-cipher) of Up -> Left on the characters: `8uq2dge858649372`

Together, we get the sequence `CBVBBMVCNB8uq2dge858649372`

If you'd like to solve the 7-segmented displays more easily, convert all characters in the sequence (and image) to uppercase or lowercase, and use [geocachingtoolbox.com](https://www.geocachingtoolbox.com/index.php?lang=en&page=segmentDisplay) to get each character of the flag.

Now we color the segments of character that contain a character in the sequence in the color red.

![SolvedSegments](/assets/img/posts/solvedsegments.png){: w="958" h="312" }

Revealing the flag `jolly`.

## Day 21: Head(set) Hunt!

This day's puzzle contains a link to the website [https://sentor.djul.datasektionen.se](https://sentor.djul.datasektionen.se/), containing a loginform and a hint towards the website's availability through a Tor hidden service.

![Sentor](/assets/img/posts/sentor.png){: w="822" h="473" }


Snooping around in the Chrome Debugger, I noticed that assets were being served from the `files` folder.

![Files](/assets/img/posts/files.png){: w="246" h="126" }

Navigating to [https://sentor.djul.datasektionen.se/files/](https://sentor.djul.datasektionen.se/files/) shows the index of `/files`, revealing more files than loaded by the index page.

![Index](/assets/img/posts/index.png){: w="466" h="322" }

`todo.txt` contains the following with markdown checkbox syntax. The star marks which tasks have been completed by the developer.

```markdown
[*] change admin password to "bomberman"
[*] learn how to exit vim:wqq:qq:w
[ ] fix directory listing, but how????
[ ] I lost my tor server files, and could only recover the ed25519 public key. The tor server is still up and running, but I lost the hostname too. Call IT support, perhaps they can recover it...
```

Although it would seem to be too easy, using the credentials `admin` and `bomberman` on the website, we get the form validation message

```
ERROR: user login not permitted from the clearnet. Go away!
```

We therefore have to complete this developer's work by deriving the tor hidden service hostname from an ed25519 public key. Conveniently, a .zip file is hosted in the image above containing the following files.

| Filename                | Size     | Contents |
| ----------------------- | -------- | ---------|
| `hs_ed25519_public_key` | 64 Bytes | Binary   |
| `hs_ed25519_secret_key` | 1  Byte  | Empty    |
| `hostname`              | 0  Bytes | Empty    |

> The contents of the public key contain strange characters as it must be opened in binary format for reading.
{: .prompt-info }


In true CTF-style, we are not going to reverse engineer the ed25519 specification. Instead, we first take way too long querying Google if somebody has done this work for us. Fortunately, I encountered a the repository [github.com/cmehay/pytor](https://github.com/cmehay/pytor) that contains a file called `onion.py` ([link](https://github.com/cmehay/pyto/blob/master/pytor/onion.py)) that seems to have functions that we need to derive the hostname. Based on the format shown in the inheritence structure, our public key's header specifies that it is an Onion V3 address.

```python
from base64 import b32encode
from hashlib import sha3_256


def get_onion_str(_pub) -> str:
    version_byte = b"\x03"

    def checksum(pubkey):
        checksum_str = ".onion checksum".encode("ascii")
        return sha3_256(checksum_str + _pub + version_byte).digest()[:2]

    return (b32encode(_pub + checksum(_pub) + version_byte).decode().lower())


with open('hs_ed25519_public_key', mode='rb') as f:
    key_data = f.read()    
    
    # Only get the relevant part by removing the header.
    key_data = key_data.replace(b"== ed25519v1-public: type0 ==\x00\x00\x00", b"")

    print(get_onion_str(key_data))
```

Running the above Python code gives us the Onion V3 address. Finally, we append `.onion`

[hw44qvorlbwlf7binb4ko4edms6wtj26dkdmju75exwmh56dnjlzylqd.onion](https://hw44qvorlbwlf7binb4ko4edms6wtj26dkdmju75exwmh56dnjlzylqd.onion/)

Navigating to this hidden server using the Tor browser, we enter the username `admin` and password `bomberman` to reveal the prompt.

> Welcome admin - the flag is tabletennis

The flag thus being `tabletennis`.

## Day 22: Virtual Pilot

This day's puzzle text instructs us to do the following

> What you need to do is find an evil duck somewhere - I think it’s hidden somewhere not visible on the map.

and

> “It’s like I always say, **N**ever **O**pen **S**tyled **Q**uacking **L**emons - make lemonade with them instead.” dAnkan says, sounding very satisfied with the idiom.

With the capitalization referring to the use of NoSQL for this puzzle.

The text then links us to [cparta.djul.datasektionen.se](https://cparta.djul.datasektionen.se/), which contains an OpenStreetMap with a set of lakes containing ducks around the world. Where are these ducks coming from though?

Opening the network tab in the Chrome Debugger shows a GraphQL endpoint [cparta.djul.datasektionen.se/graphql](https://cparta.djul.datasektionen.se/graphql). Navigating to this URL shows us the Apollo Studio portal. This enables us to skip [introspection](https://graphql.org/learn/introspection/) steps and manually firing request via e.g. Postman.

![Apollo](/assets/img/posts/apollo.png){: w="471" h="590" }

Clicking on `Query your server` brings us to [studio.apollographql.com/sandbox/explorer](https://studio.apollographql.com/sandbox/explorer) with the sandbox URL specified to `https://cparta.djul.datasektionen.se/graphql`.

Going from the root, we encounter 4 types of queries that can be performed

* `countries: [Country]` with no arguments
* `country: Country` with the argument `query: JSON!`
* `duck: Duck` with the argument `id: ID!`
* `pond: Pond` with the argument `name: String!`

We can also learn from the queries that are already being performed by the front-end itself. Inspecting the Network tab in the Chrome Debugger, identifiying the request, and selecting the payload tab when clicking on a random pond, reveals the following GraphQL query.

```graphql
query Pond($name: String!) {
    pond(name:$name) {
        ducks {
            ... on FancyDuck {
                id
                name
                family {
                    name
                }
                color
            }
            ... on Mallard {
                id
                name
                family {
                    name
                }
                ability
            }
        }
    }
}
```
Where `$name` is e.g. `Vänern`

We're looking for an evil duck. What can we get to know about ducks?

The introspection feature in Apollo Studio shows that there's a type `EvilDuck` with the fields
* `family: [Duck]!`
* `flag: String!`
* `id: ID!`
* `name: String!`

The flag, exactly what we're looking for.

The puzzle text specified that it would involve NoSQL. The only place where that might occur is the `JSON!` argument for the Country query. This is (probably) backed by MongoDB on the server.

Partly inferred from the example query above, we construct a new query. Apollo Server also provides code completion for the available fields by pressing `Ctrl + Space`.

```graphql
query Country($query: JSON!) {
    country(query: $query) {
        ponds {
            ducks {
                ... on EvilDuck {
                    id
                    name
                    family {
                        id
                        name
                    }
                    flag
                    __typename
                }
            }
        }
        name
    }
}
```

Now we need to know what to query for. After much trial-and-error and a hint from another participant that would like to stay anonymous, I came to the conclusion that I'd have to query ducks that are not in the set of countries known to us. This is mainly because the 'country query' cannot return ducks within multiple countries. It will always default to the first index in the result set, in order, which is Sweden.

Writing the `$query` variable thus becomes

```json
{
    "query": "{\"name\": {\"$nin\": [\"Norway\", \"Sweden\", \"Denmark\", \"United States\", \"Canada\", \"Germany\", \"Ireland\", \"Scotland\", \"England\", \"India\", \"China\", \"Ukraine\", \"Madagascar\", \"Colombia\", \"Brazil\", \"Peru\"]}}"
}
```

Giving us the result

```json
{
  "data": {
    "country": {
      "ponds": [
        {
          "ducks": [
            {
              "id": "5592ad76-d26c-48ec-9d75-54ddcfd4045b",
              "name": "Dark Lord of Aitken basin",
              "family": [
                {
                  "id": "0f055c94-53cd-4990-bc87-daadc896de48",
                  "name": "Satan"
                }
              ],
              "flag": "monstrosity",
              "__typename": "EvilDuck"
            }
          ]
        }
      ],
      "name": "Moon"
    }
  }
}
```

The flag thus being `monstrosity`.

### Additional insights 

This puzzle took me way longer than the previous ones. A major waste of time was caused by being unsuccessful to write queries like

```json
{
    "query": "{\"ponds.ducks\":{\"$elemMatch\":{\"__typename\":\"EvilDuck\"}}}"
}
```

and the subsequent backtracking after (because I thought I missed something). After solving the puzzle, I asked this in the Discord channel. The following answer was provided.

> "A query like that isn't possible because MongoDB doesn't actually store the ponds in the countries and the ducks in those ponds. Everything is referred to by their respective IDs and resolved properly on the backend. In this case `ponds.ducks` wouldn't make much sense, since ponds is a list of IDs (which on the backend is turned in to a list of type `Pond`)"  -- nnewram, cparta (sponsor) on Discord

Having barely touched MongoDB and GraphQL in the past, we learned something!.

## Day 23: Pulling the plug

The last puzzle that counts towards points for the leaderboard is Day 23. This puzzle is another MUD game. This time, however, you're able to specify your seperate room name that shouldn't be easy to guess by another participant, indicating that you're able to connect to the room with multiple netcat connections by having multiple terminal windows open.

The map consits of a number of rooms that _can_ contain one of the following entities.
* _Trap door_, which opens and remains open in the room (i.e. makes the unaccessible again) when a player leaves the room after triggering the trapdoor.
* _Door_, which can be in the state '_Open_' or '_Closed_'.
* _Gate_, which only opens when all trap doors are opened
* _Pressure plate_, which flips the state of ALL doors either '_Open_' or '_Closed_'. (similar to a boolean negation)

I then tested the multi-user functionality, and found the following constraints
* At most 3 terminals can be connected at once to a single dungeon instance.
* `Multiple players can only be in the start and ending rooms!`
* `There are players inside door rooms. Going on the plate would crush someone, and we cant have that.`
* Disconnecting and reconnecting leaves the room state in tact, but returns that player to the start ($S$) position. (Hinted to me by anonymous participant)

Below is a map of this dungeon.

![Dungeon](/assets/img/posts/bigmap.png){: w="1123" h="525" }

* The red colored squares, containing the character `T` indicates a room with a _Trapdoor_
* The brows colored squares, containing the character `D`, and their respective initial states `O` for _Open_ or `C` for _Closed_ as subscript, indicate a room with a _Door_.
* The golden colored squares, containing the character `G` indicates a room with a _Gate_. 
* The blue colored squares, containing the character `P` indicates a room with a _Pressure plate_.

The goal is to have all 3 connected clients operate a jigsaw in the Exit room to reveal the flag. In order to get three players to the exit, we have to trigger all trapdoors and juggle the Pressure plates to open and close doors.

Using the map above, _a_ solution can be achieved by following these instructions.

1. Player 1 traverses the sequence $11{\text -}5 \rightarrow 12{\text -}5 \rightarrow 12{\text -}6 \rightarrow 12{\text -}7$, triggering two trapdoors, and disconnects and reconnects.
2. Player 1 triggers a pressure plate by following the sequence $10{\text -}4 \rightarrow 10{\text -}3 \rightarrow 10{\text -}2 \rightarrow 9{\text -}2$, triggering 1 trap door.
3. Player 2 traverses the sequence $10{\text -}6 \rightarrow 10{\text -}7 \rightarrow 9{\text -}7 \rightarrow 8{\text -}7 \rightarrow 7{\text -}7$ 
   $\rightarrow 6{\text -}7 \rightarrow 5{\text -}7 \rightarrow 4{\text -}7 \rightarrow 3{\text -}7 \rightarrow 2{\text -}7 \rightarrow 1{\text -}7 \rightarrow 1{\text -}6 \rightarrow 2{\text -}6 \rightarrow 3{\text -}6 \rightarrow 3{\text -}5 \rightarrow 2{\text -}5$, triggering 10 trap doors by disconnecting and reconnecting afterwards.
4. Player 2 goes west twice by traversing $9{\text -}5 \rightarrow 8{\text -}5$.
5. Player 1 steps off $9{\text -}2$ pressure plate by going to $9{\text -}3$.
6. Player 2 again goes west twice by traversing $7{\text -}5 \rightarrow 6{\text -}5$.
7. Player 1 step back on $9{\text -}2$ pressure plate, triggering the trapdoor on $9{\text -}3$.
8. Player 2 traverses the sequence $5{\text -}5 \rightarrow 4{\text -}5 \rightarrow 5{\text -}5 \rightarrow 5{\text -}4 \rightarrow 5{\text -}3 \rightarrow 4{\text -}3 \rightarrow 5{\text -}3$ 
   $\rightarrow 6{\text -}3 \rightarrow 6{\text -}2 \rightarrow 6{\text -}1 \rightarrow 5{\text -}1 \rightarrow 4{\text -}1 \rightarrow 4{\text -}2 \rightarrow 4{\text -}1$, triggering 5 trapdoors and thus clearing them all.
9. Player 3, from the starting postiion $S$, traverses the path towards $13{\text -}2$, through the gate.
10. Player 1 disconnects and reconnects, now being at position $S$.
11. Player 3 continues towards $14{\text -}5$, triggering the pressure plate.
12. Player 1 continues towards $13{\text -}2$.
13. Player 3 goes towards the exit, leaving the pressure plate.
14. Player 1 continues towards $14{\text -}5$, triggering the pressure palte.
11. Player 2 disconnects and reconnects, now being at position $S$, and follows the paths towards the exit with the help of Player 1. The process is similar in how Player 3 helped Player 1.

After all three players interact with the `jigsaw` in the Exit room, the flag is revealed to be `prescription`

## Day 24: Not so fast!

The last puzzle (that doesn't count towards any leaderboard points) was a Google Form for providing feedback. At the end of the form, the flag was listed: `improvability`.


## Completion Times

| Day   | Completion Time     | Rank |
| ----- | ------------------- | ---- |
| -1    | day 01 at 15:37:47  | 221  |
| 01    | day 01 at 15:42:27  | 145  | 
| 02    | day 02 at 13:04:39  | 89   |
| 05    | day 05 at 20:32:47  | 101  |
| 06    | day 08 at 22:53:46  | 102  |
| 07    | day 09 at 00:01:38  | 87   |
| 08    | day 09 at 00:30:36  | 77   |
| 09    | day 11 at 22:21:44  | 58   |
| 12    | day 12 at 16:08:33  | 59   |
| 13    | day 13 at 13:49:16  | 39   |
| 14    | day 14 at 19:50:38  | 44   |
| 15    | day 15 at 12:48:55  | 7    |
| 16    | day 16 at 14:47:06  | 43   |
| 19    | day 19 at 12:59:47  | 6    |
| 20    | day 20 at 21:16:47  | 31   |
| 21    | day 21 at 21:01:17  | 27   |
| 22    | day 23 at 01:16:04  | 19   |
| 23    | day 23 at 19:25:51  | 37   |
| 24    | day 24 at 12:55:50  | 12   |
|       |                     |      |
| Final |                     | 35   |  
