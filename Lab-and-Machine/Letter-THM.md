# Letter

[TryHackMe room](https://tryhackme.com/room/letter)

The room gives us three files, `letter.png`, `Newspaper_clipping.png`, and `Note.txt`. We need to find a postal code and the name and age of the person mentioned in the note.

## 1. Postal code

The envelope is addressed to **Édouard G.** It also shows **SNSM**, the French sea-rescue organisation. Most of the address is damaged, so the newspaper is a useful place to look for the location.

The newspaper is **L'Ouest-Éclair**. Part of the main headline is missing, but **Finistère** and **canots de sauvetage** are visible. The second phrase means rescue boats.

Search for these words together.

```text
"L'Ouest-Éclair" "Finistère" "canots de sauvetage"
"Finistère" "catastrophe" "deux canots"
```

The results point to a disaster in **Penmarc'h on 23 May 1925**. Two fishing boats were in trouble near Kérity. Two rescue boats went out to help, but their crews also ended up in the water. Other fishing boats joined the rescue.

The account on KBC Penmarc'h describes the event. The place and the two rescue boats match the newspaper clues. [KBC Penmarc'h](https://kbcpenmarch.franceserv.com/la-catastrophe-du-23-mai-1925-selon-les-annales-du-sauvetage.html).

This gives us a town to check. Search for its postal code, using **Penmarch**, the spelling also used in postal addresses.

```text
Penmarch code postal La Poste
```

La Poste lists the town as **PENMARCH (29760)**. The answer to the first question is **29760**. [La Poste](https://localiser.laposte.fr/finistere/penmarch).

## 2. Name and age

The note talks about Édouard's **great-grandfather**. It calls him **le benjamin de l'équipe**, which means the youngest member of the team. It also says he was too young to take a driving test when he helped that day.

The next step is to look for a young member of a crew that helped during the disaster.

```text
"Penmarc'h" "1925" "sauveteurs" "benjamin"
"Penmarc'h" "1925" "mousse"
```

The French word **mousse** means cabin boy, a young member of a ship's crew.

Open the Infobretagne page and search within it for `mousse`. The list of people who received medals includes **Yves-Marie Gourlaouen**, aged **15**, from the **Arche d'Alliance**. He received a second-class silver medal for his part in the rescue. His age and medal fit the note's clues. [Infobretagne](https://www.infobretagne.com/penmarch-naufrages.htm).

The Papa Poydenot website also has an account called **Témoignage d'Yves Gourlaouen**. In it, he describes working as a cabin boy on the same boat during the disaster. This helps confirm the connection. The full name and age are given in the Infobretagne record. [Papa Poydenot](https://www.lepapapoydenot.com/archives/drame-1925).

The person is **Yves-Marie Gourlaouen**, and he was **15 years old**. His surname also starts with the **G.** shown on the envelope.

## 3. Flag

Use the format `THM{Name_Surname_age}`. Keep the hyphen in **Yves-Marie** and use underscores between the name, surname, and age.

```text
THM{Yves-Marie_Gourlaouen_15}
```
