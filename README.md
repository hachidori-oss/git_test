# git_test
"Hello Odin!"

## How to save my AbiWord and Anki files to Github
1. Adding AbiWord Files to GitHub

Since .abw files are technically text (XML), they are actually very "Git-friendly."

    Move the file: Ensure your Fixed_Anki_questions.abw is inside your local repository folder (e.g., ~/Documents/odinproject/my-repo/).
cd ~/Documents/odinproject/my-repo/
git add Fixed_Anki_questions.abw
git commit -m "Add flashcard notes from AbiWord"
git push origin main

2. Finding and Exporting Anki Flashcards

Anki doesn't store cards as individual files in a folder; it uses a hidden database. To get them into GitHub, you have to export them.
A. Exporting the "Anki Way" (Manual) = All Images will be lost!

    Open Anki.

    Click the gear icon next to the specific deck you want to save.

    Select Export...

    Export Format: Choose Notes in Plain Text (.txt).

        Why? Plain text is much easier to read on GitHub than the .apkg format (which is a compressed binary).

    Include: Select your specific deck.

    Save location: Save this .txt file directly into your GitHub repository folder.

B. The .apkg (Portable Backup) = Everything will be saved (Images and deck progress)
    This is the easiest "all-in-one" method. It bundles your cards, formatting, and images into a single file.

    In Anki: File > Export.

    Select Anki Collection Package (.colpkg) if you want the whole database, or Anki Deck Package (.apkg) for just one deck.

    Crucial: Ensure the box "Include media" is checked.

    Save this file into your local GitHub repo folder.

    Terminal:
        ```
        git add your_deck_name.apkg
        git commit -m "Backup Anki deck with images"
        git push origin main
        ```

## What to do when there is an "Invalid Document" error for an AbiWord file Part? Part 2
Now that you have a repo, you can use a "safety first" habit:

   1.  Save your work in AbiWord.

   2.  In the terminal, run git diff.

   3.  If you see thousands of lines of props="" props="" (like we found today), don't commit it. Close AbiWord and try to save again, or run your sed fix.

   4.  If the diff looks clean (just your new text), git commit and push.


## What to do when there is an "Invalid Document" error for an AbiWord file? Part 1
Error: "AbiWord cannot open -location of the file-. It appears to be an invalid document."

The most likely culprit is a XML syntax error. An .abw file is essentially a text file written in XML format. If a single character was misplaced during the last save, or if there was a "bit flip" on your hard drive, AbiWord might fail to parse the entire document.

Since .abw is just XML, you can often open it in a plain text editor to see what’s wrong.
1. Right-click the file and select Open With -> Text Editor (Xed) or Gedit.

2. If the file opens and you see a bunch of code starting with `<abiword ...>`, congratulations! Your data is still there.

3. Scroll to the very bottom of the file. A common issue is that the file didn't finish saving, so the "closing tags" are missing. It should end with:
`</abiword>`

4. If it doesn't, add that tag, save it, and try opening it in AbiWord again.

5. Use xmllint to find the error
    Linux Mint usually comes with libxml2-utils. You can ask the system exactly where the "invalid" part of the document is:

    Open your terminal.

    Type: xmllint /path/to/your/file.abw

    The terminal will spit out an error message telling you exactly which line and column is breaking the document. You can then go to that line in your text editor and fix the typo.

6. I typed `file Desktop/Potential\ Anki\ questions.abw ` into the terminal and got back `Desktop/Potential Anki questions.abw: symbolic link to /home/haru/Documents/odinproject/Potential Anki questions.abw` 
    Ah, that explains a lot! What you have on your Desktop isn't the actual file; it's a symbolic link (basically a shortcut) pointing to a file in your Documents folder.

    The error "invalid document" often happens when a shortcut is "broken" or pointing to a file that has been moved, renamed, or—more likely in this case—the destination file itself is the one that's corrupted.
7. After typing `file /home/haru/Documents/odinproject/Potential\ Anki\ questions.abw` in the terminal I got back `/home/haru/Documents/odinproject/Potential Anki questions.abw: XML 1.0 document, Unicode text, UTF-8 text, with very long lines (1846)` 
    hat is actually great news! The fact that the system identifies it as "XML 1.0 document" means the file is not "dead" or empty; it’s just "broken" in a way that AbiWord’s strict parser can’t handle.

    Since you are seeing "very long lines (1846)", it suggests the entire document (or a huge chunk of it) is crammed onto a single line. If there is a single stray character or a tag that didn't close properly on that long line, AbiWord will panic and call it "invalid."

    Here is your recovery plan, step-by-step:
        Use the "Formatting" trick

        If the file is hard to read because of those "very long lines," you can use a tool to make it human-readable again. In your terminal, run:
        `xmllint --format /home/haru/Documents/odinproject/Potential\ Anki\ questions.abw > recovered_file.abw`

        This takes your messy, one-line file and "pretty-prints" it into a new file called recovered_file.abw.

        If xmllint encounters an error, it will tell you exactly which line is the problem.
            Step 1: Install the repair tool

            First, let's get the tool that will tell us exactly where the "invalid" part is. Open your terminal and run:

            `sudo apt update && sudo apt install libxml2-utils -y`

            (You’ll need to type your password; characters won't show up as you type, which is normal for Linux).
            Step 2: Identify the error

            Now, run the command again, but we will look at the error output this time:

            `xmllint --noout /home/haru/Documents/odinproject/Potential\ Anki\ questions.abw`

                After typing `xmllint --noout /home/haru/Documents/odinproject/Potential\ Anki\ questions.abw` in the terminal I got back:
                    `/home/haru/Documents/odinproject/Potential Anki questions.abw:277: parser error : Attribute props redefined -position:normal; font-stretch:normal"> </c><c props="" props="font-weight:bold"  /home/haru/Documents/odinproject/Potential Anki questions.abw:277: parser error : Attribute props redefined y:Inter Variable; text-transform:none; font-size:16.000000pt; font-style:normal" ...

                        This is the "smoking gun." The error Attribute props redefined means that AbiWord somehow wrote the same setting twice in one tag, which is illegal in XML.

                        Specifically, it wrote props="" props="font-weight:bold". XML rules say you can only have props once per tag. Because this is happening on line 277 (which is likely a massive, long line), it's failing over and over.
                            Option 1: The Automated Terminal Fix (Recommended)

                            Since the error is very specific, we can use a command to "find and replace" the broken part automatically. Run this command in your terminal:

                            sed 's/props="" props=/props=/g' /home/haru/Documents/odinproject/Potential\ Anki\ questions.abw > /home/haru/Documents/odinproject/Fixed_Anki_questions.abw

                            What this does:

                                It searches the file for every instance of the double-definition props="" props=.

                                It replaces it with just a single props=.

                                It saves the result as a new file called Fixed_Anki_questions.abw.
                            
                            Option 2: The Text Editor Manual Fix

                                If the command above doesn't catch everything, you can do it manually in a text editor (like Xed):

                                    Open the file in your Text Editor.

                                    Press Ctrl + H (Replace).

                                    In the Search for box, type: props="" props=

                                    In the Replace with box, type: props=

                                    Click Replace All.

                                    Save the file and try opening it in AbiWord.

8. Extracting text without the "Formatting Junk"

    If you just want your questions back without the XML tags (the <p>, <section>, and <styles>), you don't have to copy-paste manually. Linux has a tool called sed that can strip away the code for you.

    Run this command to create a "Clean" text-only version of your file:

    `sed 's/<[^>]*>//g' /home/haru/Documents/odinproject/Potential\ Anki\ questions.abw > /home/haru/Documents/odinproject/Extracted_Text.txt`

    What this does:

        sed 's/<[^>]*>//g': This looks for anything inside <brackets> and deletes it.

        > Extracted_Text.txt: This saves the result into a plain text file.

        You can then open Extracted_Text.txt with your Text Editor. All your questions will be there, though you’ll need to delete some empty lines at the top where the headers used to be.