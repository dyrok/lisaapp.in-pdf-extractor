# Lisa Notes Extractor — README

![Pi7_GIF_CMP](https://github.com/user-attachments/assets/97785051-2f0e-4861-8e8b-a1de98d29ccb)

## Installation

1. Install **Tampermonkey** or **Violentmonkey** extension in your browser.
2. Create a new userscript.
3. Copy and paste the full script code (including the header).
4. Save. The script will run automatically on all pages.

## Quick Usage

* Visit any web page.
* Press **Ctrl + Alt + D** to open the extractor popup.
* You’ll see a neat centered window listing all PDFs or Google Docs viewer links found on that page.
* Click **Copy** to copy a link to your clipboard.
* Click **Open** to open a link in a new tab.
* Use the toggle icons in the popup to switch dark/light mode or show/hide the floating button.

On Lisa App pages, small “📄 PDFs” buttons will also appear next to each container for quick access.

## How It Works

* The script scans the current page for links ending in `.pdf`, Google Docs `gview` viewer URLs, or any `/pdf` pattern.
* It groups similar links together so you don’t get duplicates.
* When you hit the shortcut (or click the floating button), it shows a clean shadcn-inspired popup listing the found links.
* All settings (dark mode, floating button visibility, which groups you expanded) are saved so they stay the same next time.

That’s it — no extra setup. Just install, visit a page with PDFs, hit the shortcut, and grab your links instantly.
