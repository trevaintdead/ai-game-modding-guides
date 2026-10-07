# Legal Notice

**This is not legal advice and it does not create any rights.** Nobody reading this is protected by it. It exists to record what this project is, to point at the behaviour that genuinely lowers your risk, and to set out the process that applies if a publisher comes after you.

If you have money, the single best thing you can do is talk to an IP lawyer before you publish, not after you receive a letter. Everything below is general information.

## What this repository is

A set of community-written guides to using AI coding agents for personal game modding. The intended scope throughout is:

- Games **you own a copy of**, on your own hardware
- **Single-player or offline** play
- Mods and tools that **read from your own install** rather than shipping game content
- Projects released as **source code only**, under an open licence, credited to whatever they were built on

The guides are **unofficial fan material**. They are not affiliated with, endorsed by, or sponsored by any game's developer or publisher. No trademark of any game publisher is used other than to identify the game being discussed.

## What a disclaimer can and cannot do

Being upfront about scope does not make unlawful conduct lawful, and this document will not stop a publisher from contacting you. It does two things that have some legal weight:

**It is evidence against willfulness.** US copyright law allows enhanced damages for willful infringement, and statutory damages for a single work can reach $150,000 where infringement is willful. Courts have entered judgments at exactly that per-work maximum. A documented, good-faith position on what your project does and does not do is evidence that you were not trying to get something you knew you had no right to. That is the only place where writing things down matters legally.

**It sets expectations for readers.** Someone who knows the scope before they clone something is less likely to be surprised by it.

That is the whole of it. Everything below is about behaviour, because behaviour is what actually determines your exposure.

## What actually lowers your risk

In rough order of how much it matters:

1. **No game files in your repository.** Not assets, not textures, not audio, not maps, not decompiled source, not a game dump. Code, docs, and build scripts only. This is the one that ends hobby projects. Use a whitelist `.gitignore` and check your history with `git log --all --stat`, because an asset committed once and deleted later is still in the history and still public.
2. **Do not circumvent anything.** Using a loader the publisher ships, or the community maintains openly, is a different act from patching a check out of an executable. Anti-cheat is the same category. Circumvention is treated more seriously than copying, and the 2026 Switch litigation was decided on exactly that basis: in September 2026 a federal court entered an injunction against vendors selling MIG Switch flash cards and dumpers, on the grounds that the products themselves circumvented Nintendo's technological protection measures. A default judgment of $4.5 million against an r/SwitchPirates moderator in the same campaign was calculated on the same theory. Nothing in this repo helps you do either of those things.
3. **Do not compete with the original.** A reimplementation aimed at interoperability is in a different position from a competing product. The US cases that protect reverse engineering are all about understanding and compatibility.
4. **Do not copy the original's quirks.** Identical bugs and identical dead code are treated as strong evidence of copying. Write your own.
5. **Keep records.** Receipts for the games you own, the versions you tested against, dates. If someone alleges you were distributing, your own records are what you produce.
6. **Credit upstream projects** and keep their licences. See [guide 6](guides/06-rules-legal-and-publishing.md).
7. **Say what you used AI for.** Several projects in this community do, and undisclosed AI-generated releases get a hostile reception in some circles.
8. **Stay offline for anything with anti-cheat.** Never build or publish anything that helps someone bypass it, including instructions.

## If your repository gets a takedown

This happens even without shipping assets. Take-Two had GitHub remove re3 and reVC and later sued the authors. Activision sent a cease-and-desist to the H2M mod the day before launch. Garry's Mod removed twenty years of Nintendo-related Workshop content after a takedown request.

GitHub and Steam are "service providers" under 17 U.S.C. § 512, so the DMCA notice-and-counter-notice process applies to material they remove. This is the part that gives you a formal route rather than just deleting and hoping.

**Do not ignore it, and do not immediately argue.** Start by working out whether you agree with the claim. Material that should not have been there comes down, with a short note saying so, and then you move on. Contesting a claim you know is valid costs more than removing the file.

A claim you think is wrong has a formal answer. § 512(g)(3) sets out what a counter-notification must contain:

- Your physical or electronic signature
- Identification of the material that was removed and **where it appeared** before removal
- A statement **under penalty of perjury** that you have a good faith belief the material was removed as a result of mistake or misidentification
- Your name, address and telephone number, plus a statement **consenting to the jurisdiction** of the Federal District Court for the district where you live (or, outside the US, any district where the platform can be found), and that you will accept service of process

Then, under § 512(g)(2), the platform must replace the material **not less than 10 and not more than 14 business days** after receiving your counter-notice, unless the original complainant tells the platform they have filed a court action.

Three warnings, and they are not small:

**Filing a counter-notice is a legal filing, not a support ticket.** It carries a perjury attestation.

**You accept being sued in the process.** The consent to jurisdiction and service of process is what gives the statement its force. The plaintiff has ten to fourteen business days to decide whether to file.

**A knowingly false counter-notice creates your own liability.** § 512(f) makes anyone who knowingly materially misrepresents that material is infringing, or that removal was a mistake, liable for damages and attorneys' fees incurred by the parties who relied on it.

**Get a lawyer before you file one.** A few hundred dollars of IP advice is worth a great deal here compared with the exposure of a perjury-attested filing.

## If you receive a cease-and-desist letter

Do not reply substantively on your own. Do not agree to anything, and do not ignore it.

- Note the deadline and calendar it. Responders get more time than people expect.
- Get an IP lawyer. This is the point where it stops being a hobby matter.
- Do not delete your repository in a panic, and do not wipe your git history, without advice. Deleting can look like destruction of evidence, and the history is your evidence of what you did and did not ship.
- Gather the records listed under "What actually lowers your risk" above: receipts, versions, dates, your own commit history, your `.gitignore`.

Most cease-and-desist letters end without a lawsuit. Some do not. The variable that matters most is whether you handled it carefully at the start.

## Reuse

The guides are MIT licensed. The legal analysis in [guide 13](guides/13-reverse-engineering-and-the-law.md) is educational and summarises publicly available case law and statute. It is not a substitute for advice about your specific project, and anyone reusing it should say so themselves.

Cases are from the United States. The EU and UK treat reverse engineering more narrowly in some respects; guide 13 covers the difference. If you are outside the US, that section matters more than the rest of this file.

## Further reading

- [guide 6](guides/06-rules-legal-and-publishing.md), the practical rules
- [guide 13](guides/13-reverse-engineering-and-the-law.md), how the law actually treats reverse engineering
- [guide 10](guides/10-posting-your-project.md), publishing, and the pre-flight checklist
- 17 U.S.C. § 102(b), § 117, § 1201 and § 512
- The U.S. Copyright Office's [Circular 10](https://www.copyright.gov/circs/circ10.pdf) on section 512
