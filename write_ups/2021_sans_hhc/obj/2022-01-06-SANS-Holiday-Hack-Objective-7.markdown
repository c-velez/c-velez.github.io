---
layout: post
date: 2022-01-06 08:15:26 -500
categories: jekyll update
---

# Printer Exploitation

**Objective**: We need to access the printer to see what the last file that was printed was.  
**Location**: Jack Frost Tower - Jack's Office  
**Hint Terminal**: Shellcode Primer  
**Hints**:
1. Printer Firmware: When analyzing a device, it's always a good idea to pick apart the firmware. Sometimes these things come down Base64-encoded.
2. Hash Extension Attacks: [Hash Extension Attacks](https://blog.skullsecurity.org/2012/everything-you-need-to-know-about-hash-length-extension-attacks) can be super handy when there's some type of validation to be circumvented.
3. Dropping files: Files placed in `/app/lib/public/incoming` will be accessible under https://printer.kringlecastle.com/incoming/.

## Write Up

When we click the printer, we are presented with the management UI of the printer. Let's go ahead and download the firmware

![Printer Management Web Console](/assets/img/2021_sans_hhc/obj/obj07/picture_1.png){: width="750"}
<p align="center"><strong>Figure 1: Printer Management Web Console</strong></p>

![Firmware Download and Upload Page](/assets/img/2021_sans_hhc/obj/obj07/picture_2.png){: width="750"}
<p align="center"><strong>Figure 2: Firmware Page</strong></p>

The printer provides us with a JSON file with several fields in it. As hint one suggests, the firmware itself is base64 encoded. Additionally, we see that there is a signature that relates to what ever is encoded, as well as a secret length and hashing algorithm. Taking a look at hint 2, we know that this is all the information we need to conduct a hash extension attack. So we'll proceed with that attack vector

![Original Firmware Blob](/assets/img/2021_sans_hhc/obj/obj07/picture_3.png){: width="750"}
<p align="center"><strong>Figure 3: Original Firmware Blob</strong></p>

Decode the firmware field and put it in it's own file.

`echo "<base64 encoded string>" > encoded && cat encoded | base64 -d > original_firmware`

![Original Firmware Zip File](/assets/img/2021_sans_hhc/obj/obj07/picture_4.png){: width="750"}
<p align="center"><strong>Figure 4: Original Firmware Zip File</strong></p>

The `original_firmware` file that we create should contain the binary firmware. Running file against it, we see that it is a zip file. Let's rename the file and unzip it.

```
mv original_firmware firmware.zip && unzip firmware.zip
```

We can make a few educated guesses here on what the printer expects when firmware is uploaded.
1. It gave us a json, so it probably expects a json file
2. The firmware field was base64 encoded, so it is probably expecting a base64 encoded string in the firmware field
3. The json file should probably also contain a signature, a secret_length, and hashing algorithm
4. When the printer decodes the string, it is expecting a zip file, so our payload probably needs to be zipped.
5. The printer probably appends the 16 byte secret to the zip file to validate the signature.
6. The printer is probably looking for `firmware.bin` inside of the zip file.
7. The printer will execute the firmware

Knowing all of this and hint 3, let's construct our payload.

The objective requires us to get the name of the last file that was printed from the printer. The spool log can be found at `/var/spool/printer.log`.

Let's create our payload, and make it executable.

```
echo "cat /var/spool/printer.log > /app/lib/public/incoming/m4ntle_printer.log" > firmware.bin && 
chmod +x firmware.bin
```

The next step is to zip it up. Let's zip it up using another name other than `firmware.zip`, since we need `firmware.zip` for the hash extension attack.

```
zip payload.zip firmware.bin
```

Time to construct our hash_extender command. For this, reading is ABSOLUTELY FUNDAMENTAL, as there are several options that will be needed in order to craft this correctly.

**I placed the hash_extender binary in my path to simplify execution. Explanation of this is outside the scope of this write up.**

Running `hash_extender --help` we see several options:

![Hash Extender Options](/assets/img/2021_sans_hhc/obj/obj07/picture_8.png){: width="750"}
<p align="center"><strong>Figure 5: Hash Extender Options</strong></p>

The main ones we need to concern ourselves with are `--file`, `--secret`, `--append`, `--append-format`, `--signature`, `--format`, and `--out-data-format`

We want to append our zipped payload to the zipped firmware. Feeding the `secret_length` found in the original json, we use that as our input to `--secret`. We want to append the hex format of our `payload.zip`. `-p` removes the offsets of the hexdump and `-c 999999999` ensures that the output is continuous without newline chacters. We pass hash_extender the original signature so it can match the length of the secret. We pass it the hashing algorithm we found in the original json, and we want our output to be in hex form.

```
hash_extender --file firmware.zip --secret 16 --append `xxd -p -c 9999999999 payload.zip` 
--append-format hex --signature 2bab052bf894ea1a255886fde202f451476faba7b941439df629fdeb1ff0dc97 
--format sha256 --out-data-format hex
```

Running the above results in:

![Hash Extender Output](/assets/img/2021_sans_hhc/obj/obj07/picture_9.png){: width="750"}
<p align="center"><strong>Figure 6: Hash Extender Output</strong></p>

Time to build our json

First we need to encode our new string. Let's use the Cyber Chef recipe From Hex to Base64. Something to note is that are new base64 encoded string is eerily similar to the original base64 encoded string. This is because we are just appending data to the end of the original file we received.

![Cyber Chef Recipe](/assets/img/2021_sans_hhc/obj/obj07/picture_10.png){: width="750"}
<p align="center"><strong>Figure 7: Cyber Chef Recipe</strong></p>

The base64 output from Cyber Chef will live in the firmware field of our crafted json and the new signature generated from our `hash_extender` command will live in the signature field of our crafted json. Secret length and algorithm will be the same as the original json.

![Original to New Comparison](/assets/img/2021_sans_hhc/obj/obj07/picture_11.png){: width="750"}
<p align="center"><strong>Figure 8: Comparing the Original JSON Blob to Our Crafted JSON Blob</strong></p>

Time to see if this actually worked! Let's go ahead and upload our crafted json to the printer.

It looks like it landed!

![Upload Successful](/assets/img/2021_sans_hhc/obj/obj07/picture_13.png){: width="750"}
<p align="center"><strong>Figure 9: Upload Successful</strong></p>

Browsing to http://printer.kringlecastle.com/incoming/m4ntle_printer.log we see that script executed successfully!

![Printer.log](/assets/img/2021_sans_hhc/obj/obj07/picture_1.png){: width="750"}
<p align="center"><strong>Figure 10: Printer.log</strong></p>

Answer: Troll_Pay_Chart.xlsx

---
<p align="center"><a href="/write_ups/2021_sans_hhc/obj/2022-01-06-SANS-Holiday-Hack-Objective-6">< Objective 6</a> :: <a href="/write_ups/2021_sans_hhc/obj/2022-01-06-SANS-Holiday-Hack-Objective-8">Objective 8 ></a></p>