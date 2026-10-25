# Cryptography Project: Image Steganography

A small web demo of **image steganography**, the practice of hiding a message inside an image so that nobody notices it is there. We are building it for the Cryptography and Security course (academic year 2026/27, topic 9) as a program implementation by a group of two, with evaluation due in mid-January 2027.

The demo hides a short, optionally encrypted message in an image, extracts it again, shows that the change is invisible to the human eye, and then detects it with a statistical attack.

**Status:** planning, no code yet.

## How it works

Each pixel of an image stores a red, a green and a blue value between 0 and 255. We replace the last bit of each value, the **least significant bit (LSB)**, with one bit of the message. This changes a value by at most 1, for example a red value of 154 (`10011010`) becomes 155 (`10011011`), which no human eye can see.

| Property | Value |
|---|---|
| Bits hidden per pixel | 3, one per color channel |
| Largest change to a color value | ±1 out of 255 |
| Capacity of a 1,000 × 1,000 image | about 375 KB, far more than a short message needs |
| Expected peak signal-to-noise ratio (PSNR) at full capacity | about 51 dB, above the 40 dB threshold for invisible changes |

The PSNR value is theoretical rather than measured: on average half of the LSBs already match the message bits, so a full image gives a mean squared error (MSE) of 0.5 and a PSNR of 10 · log10(255² / 0.5) ≈ 51.1 dB. Both measures are explained in step 3 of the plan.

## Plan

1. **Hide and extract.** We embed a text message in the LSBs and read it back. We store the message length in the first 32 bits, so extraction knows when to stop. We use PNG images, because lossy JPEG compression would destroy the hidden bits. We embed only in fully opaque pixels, because the browser canvas rounds the color values of transparent pixels, which would destroy the hidden bits as well. The message is plain text, preferably funny, such as *"The cake is a lie."*.
2. **Encrypt.** If the user enters a password, we encrypt the message with the Advanced Encryption Standard (AES) before embedding it, using the Web Crypto API that is built into every modern browser, so no extra library is needed. Steganography hides that a message exists, while encryption hides what it says. An encrypted message is therefore unreadable without the password, but the attack in step 4 still detects it.
3. **Show it is invisible.** We place the original and the stego image side by side, where they look identical. We back this with two standard measures: the MSE, which is the average squared difference between the pixel values of the two images, and the PSNR, which compares that error with the largest possible pixel value on a logarithmic scale in decibels. A PSNR above about 40 dB is commonly considered invisible. An amplified view of the LSB plane then makes the hidden data visible where the eye cannot see it.
4. **Attack.** We run the **chi-square attack**, which detects LSB embedding statistically. Embedding makes the counts of each pair of values, such as 154 and 155, nearly equal, which rarely happens in natural images. A single test over the whole image would miss a short message, because the few changed values disappear among millions of unchanged ones. We therefore test growing portions from the start of the image, the first 1%, the first 2% and so on, as in the original attack. The demo plots the detection probability against the tested portion, which stays high where the message sits and drops where it ends, compared with a low curve for the clean image. Our short message fills only about 60 pixels, while 1% of a 1,000 × 1,000 image is already 10,000, so for the attack demo we hide a long text, such as a book chapter of about 75 KB, which fills about a fifth of the image.

## Web UI

The demo is a static web page hosted on GitHub Pages. GitHub Pages serves only static files, so all processing runs in the browser with JavaScript and the HTML canvas, rather than on a Python server. This also means uploaded images never leave the user's computer.

| Tab | You give it | You get back |
|---|---|---|
| **Hide** | A PNG image, a message and an optional password | The stego image to download |
| **Extract** | A stego image and the password, if one was used | The hidden message |
| **Analyze** | The original and stego image from the Hide tab, or one uploaded image for the chi-square attack alone | MSE, PSNR, the LSB plane view and the chi-square detection plot |

## Scope

The first version covers the simple and medium features, which together already show hiding, encryption, invisibility and detection:

- **Hide and extract.** Sequential LSB embedding of a text message with a length header, in the opaque pixels of PNG images.
- **Encryption.** Optional AES encryption of the message with a password.
- **Invisibility.** The side-by-side view, MSE, PSNR and the LSB plane view.
- **Attack.** The chi-square detection plot over growing portions of the image, for the clean and the stego image.
- **Simple flow.** The Analyze tab reuses the images from the Hide tab, so the live demo runs as hide, then analyze.
Later expansions, if time allows:

- **Randomized embedding.** Embedding at pixel positions chosen from a password, which is harder for the chi-square attack to detect, compared with sequential embedding.
- **Image in image.** Hiding a whole image inside another, as in the *Hiding an image inside another* reference.
- **Other attacks.** Stronger steganalysis methods such as RS analysis.
- **JPEG support.** Embedding in JPEG images, which needs a different method than LSB.
- **Visual polish.** A nicer design once everything works.

## Team

| Member | Role | Responsibilities |
|---|---|---|
| **Claude** | Project manager | Writes plans, sets deadlines, implements nothing and takes credit for everything |
| **Alice** | Steganographer | Embedding, extraction, encryption, and the Hide and Extract tabs |
| **Eve** | Steganalyst | MSE, PSNR, the LSB plane view, the chi-square attack and the Analyze tab |

Alice and Eve share deployment to GitHub Pages, testing on a few sample images and preparing the live demo.

## TODO

- [ ] **Overview papers (26 October 2026).** Find survey papers on image steganography and steganalysis, and add the best ones to the references.

## References

- **[Simple Image Steganography in Python](https://medium.com/hackernoon/simple-image-steganography-in-python-18c7b534854f).** Andrew Scott, HackerNoon, 2018. Explains LSB embedding, why it needs lossless formats such as PNG and how to estimate the capacity of an image.
- **[Steganography: Hiding an image inside another](https://medium.com/data-science/steganography-hiding-an-image-inside-another-77ca66b2acb1).** Kelvin Salton do Prado, Towards Data Science, 2018. Hides a whole image in the low bits of another, which shows visually why changes to the low bits go unnoticed.
- **[JavaScript Magic Tricks: Steganography](https://medium.com/@6465660/javascript-magic-tricks-steganography-a0f9330e5e0d).** LiwenWang, Medium, 2023. Implements LSB embedding in the browser with the HTML canvas, which is the approach our GitHub Pages demo uses.

## License

Released under the [MIT License](LICENSE).
