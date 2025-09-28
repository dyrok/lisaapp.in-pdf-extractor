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

## Copy and Paste the Script in tampermonkey:

```
// ==UserScript==
// @name         Lisa Notes Extractor
// @namespace    https://github.com/dyrok/
// @version      1.0
// @description  Extract PDF links and display on any page + Special Integration in Lisa (ctrl+option+p)
// @author       Neel Prabir Singh (https://github.com/dyrok/)
// @match        *://*/*
// @grant        GM_addStyle
// @grant        GM_setValue
// @grant        GM_getValue
// ==/UserScript==


(function() {
    'use strict';

    // Add Inter font and shadcn-inspired CSS with dark mode
    GM_addStyle(`
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700&display=swap');

        :root {
            --background: 0 0% 100%;
            --foreground: 222.2 84% 4.9%;
            --card: 0 0% 100%;
            --card-foreground: 222.2 84% 4.9%;
            --primary: 221.2 83.2% 53.3%;
            --primary-foreground: 210 40% 98%;
            --secondary: 210 40% 96%;
            --secondary-foreground: 222.2 84% 4.9%;
            --muted: 210 40% 96%;
            --muted-foreground: 215.4 16.3% 46.9%;
            --accent: 210 40% 96%;
            --accent-foreground: 222.2 84% 4.9%;
            --border: 214.3 31.8% 91.4%;
            --input: 214.3 31.8% 91.4%;
            --ring: 221.2 83.2% 53.3%;
            --radius: 0.5rem;
        }

        .dark {
            --background: 222.2 84% 4.9%;
            --foreground: 210 40% 98%;
            --card: 222.2 84% 4.9%;
            --card-foreground: 210 40% 98%;
            --primary: 217.2 91.2% 59.8%;
            --primary-foreground: 222.2 84% 4.9%;
            --secondary: 217.2 32.6% 17.5%;
            --secondary-foreground: 210 40% 98%;
            --muted: 217.2 32.6% 17.5%;
            --muted-foreground: 215 20.2% 65.1%;
            --accent: 217.2 32.6% 17.5%;
            --accent-foreground: 210 40% 98%;
            --border: 217.2 32.6% 17.5%;
            --input: 217.2 32.6% 17.5%;
            --ring: 224.3 76.3% 94.1%;
        }

        .pdf-extractor-dialog {
            position: fixed;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            background: hsl(var(--background));
            color: hsl(var(--foreground));
            border-radius: 12px;
            box-shadow: 0 20px 25px -5px rgba(0, 0, 0, 0.1), 0 10px 10px -5px rgba(0, 0, 0, 0.04);
            z-index: 10000;
            width: 90%;
            max-width: 700px;
            max-height: 80vh;
            overflow: hidden;
            font-family: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
            border: 1px solid hsl(var(--border));
            transition: all 0.3s ease;
        }

        .pdf-extractor-backdrop {
            position: fixed;
            top: 0;
            left: 0;
            right: 0;
            bottom: 0;
            background: rgba(0, 0, 0, 0.5);
            z-index: 9999;
            backdrop-filter: blur(8px);
            animation: backdropBlurIn 0.4s ease-out;
        }

        .pdf-extractor-header {
            padding: 1.5rem;
            border-bottom: 1px solid hsl(var(--border));
            display: flex;
            justify-content: space-between;
            align-items: center;
            background: hsl(var(--card));
        }

        .pdf-extractor-title {
            font-size: 1.25rem;
            font-weight: 600;
            margin: 0;
            color: hsl(var(--foreground));
            font-family: 'Inter', sans-serif;
        }

        .pdf-extractor-close {
            background: none;
            border: none;
            font-size: 1.5rem;
            cursor: pointer;
            padding: 0.5rem;
            border-radius: 6px;
            color: hsl(var(--muted-foreground));
            width: 40px;
            height: 40px;
            display: flex;
            align-items: center;
            justify-content: center;
            transition: all 0.2s ease;
            backdrop-filter: blur(4px);
        }

        .pdf-extractor-close:hover {
            background: hsl(var(--accent));
            color: hsl(var(--accent-foreground));
            transform: scale(1.1);
            backdrop-filter: blur(8px);
        }

        .pdf-extractor-content {
            padding: 1.5rem;
            max-height: 400px;
            overflow-y: auto;
            background: hsl(var(--background));
        }

        .pdf-categories {
            display: flex;
            flex-direction: column;
            gap: 0.75rem;
        }

        .pdf-category {
            border: 1px solid hsl(var(--border));
            border-radius: 8px;
            overflow: hidden;
            background: hsl(var(--card));
            backdrop-filter: blur(4px);
            transition: all 0.3s ease;
        }

        .pdf-category:hover {
            backdrop-filter: blur(8px);
            transform: translateY(-2px);
        }

        .pdf-category-header {
            padding: 1rem;
            background: hsl(var(--secondary));
            display: flex;
            justify-content: space-between;
            align-items: center;
            cursor: pointer;
            transition: all 0.3s ease;
            backdrop-filter: blur(4px);
        }

        .pdf-category-header:hover {
            background: hsl(var(--accent));
            backdrop-filter: blur(8px);
        }

        .pdf-category-title {
            font-weight: 600;
            color: hsl(var(--foreground));
            display: flex;
            align-items: center;
            gap: 0.5rem;
        }

        .pdf-category-count {
            background: hsl(var(--primary));
            color: hsl(var(--primary-foreground));
            padding: 0.25rem 0.5rem;
            border-radius: 20px;
            font-size: 0.75rem;
            font-weight: 500;
            backdrop-filter: blur(4px);
        }

        .pdf-category-content {
            padding: 0;
            max-height: 0;
            overflow: hidden;
            transition: all 0.4s cubic-bezier(0.4, 0, 0.2, 1);
            backdrop-filter: blur(2px);
        }

        .pdf-category-content.expanded {
            padding: 1rem;
            max-height: 300px;
            backdrop-filter: blur(4px);
        }

        .pdf-list {
            display: flex;
            flex-direction: column;
            gap: 0.5rem;
        }

        .pdf-item {
            display: flex;
            align-items: center;
            padding: 0.75rem;
            border: 1px solid hsl(var(--border));
            border-radius: 6px;
            text-decoration: none;
            color: hsl(var(--foreground));
            transition: all 0.3s ease;
            background: hsl(var(--background));
            backdrop-filter: blur(4px);
        }

        .pdf-item:hover {
            border-color: hsl(var(--primary));
            background: hsl(var(--accent));
            transform: translateY(-2px) scale(1.02);
            backdrop-filter: blur(8px);
            box-shadow: 0 10px 25px -5px rgba(0, 0, 0, 0.1);
        }

        .pdf-icon {
            margin-right: 0.75rem;
            font-size: 1.25rem;
            filter: blur(0.5px);
            transition: all 0.3s ease;
        }

        .pdf-item:hover .pdf-icon {
            filter: blur(0px);
            transform: scale(1.1);
        }

        .pdf-info {
            flex: 1;
            min-width: 0;
        }

        .pdf-filename {
            font-weight: 500;
            margin-bottom: 0.25rem;
            color: hsl(var(--foreground));
        }

        .pdf-url {
            font-size: 0.875rem;
            color: hsl(var(--muted-foreground));
            word-break: break-all;
            opacity: 0.8;
        }

        .pdf-actions {
            display: flex;
            gap: 0.5rem;
            margin-left: 0.75rem;
        }

        .pdf-action-btn {
            padding: 0.375rem 0.75rem;
            border: 1px solid hsl(var(--border));
            border-radius: 6px;
            background: hsl(var(--background));
            cursor: pointer;
            color: hsl(var(--foreground));
            transition: all 0.3s ease;
            font-size: 0.75rem;
            font-weight: 500;
            backdrop-filter: blur(4px);
        }

        .pdf-action-btn:hover {
            background: hsl(var(--accent));
            border-color: hsl(var(--primary));
            transform: scale(1.05);
            backdrop-filter: blur(8px);
        }

        .empty-state {
            text-align: center;
            padding: 3rem 2rem;
            color: hsl(var(--muted-foreground));
        }

        .empty-state-icon {
            font-size: 3rem;
            margin-bottom: 1rem;
            opacity: 0.5;
            filter: blur(1px);
            animation: gentlePulse 3s ease-in-out infinite;
        }

        /* Default floating button style - HIDDEN BY DEFAULT */
        .pdf-extractor-button {
            position: fixed;
            top: 20px;
            right: 20px;
            z-index: 10000;
            background: hsl(var(--primary));
            color: hsl(var(--primary-foreground));
            border: none;
            border-radius: 8px;
            padding: 12px 16px;
            font-size: 14px;
            font-weight: 600;
            cursor: pointer;
            box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1), 0 2px 4px -1px rgba(0, 0, 0, 0.06);
            transition: all 0.4s cubic-bezier(0.4, 0, 0.2, 1);
            font-family: 'Inter', sans-serif;
            display: flex;
            align-items: center;
            gap: 0.5rem;
            display: none;
            backdrop-filter: blur(8px);
        }

        .pdf-extractor-button:hover {
            transform: translateY(-3px) scale(1.05);
            box-shadow: 0 20px 25px -5px rgba(0, 0, 0, 0.1), 0 10px 10px -5px rgba(0, 0, 0, 0.04);
            backdrop-filter: blur(12px);
        }

        /* Lisa app specific button style - VISIBLE BY DEFAULT on Lisa app */
        .pdf-extractor-button-lisa {
            background: hsl(var(--primary));
            color: hsl(var(--primary-foreground));
            border: none;
            border-radius: 6px;
            padding: 8px 12px;
            font-size: 12px;
            font-weight: 500;
            cursor: pointer;
            transition: all 0.3s ease;
            font-family: 'Inter', sans-serif;
            display: inline-flex;
            align-items: center;
            gap: 0.25rem;
            margin: 4px;
            box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
            backdrop-filter: blur(4px);
        }

        .pdf-extractor-button-lisa:hover {
            transform: translateY(-2px) scale(1.05);
            box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
            backdrop-filter: blur(8px);
        }

        .shortcut-hint {
            font-size: 0.75rem;
            opacity: 0.9;
            font-weight: 400;
        }

        .header-actions {
            display: flex;
            gap: 0.5rem;
            align-items: center;
        }

        .theme-toggle, .button-toggle {
            background: none;
            border: 1px solid hsl(var(--border));
            border-radius: 6px;
            padding: 0.5rem;
            cursor: pointer;
            color: hsl(var(--muted-foreground));
            transition: all 0.3s ease;
            display: flex;
            align-items: center;
            justify-content: center;
            backdrop-filter: blur(4px);
        }

        .theme-toggle:hover, .button-toggle:hover {
            background: hsl(var(--accent));
            color: hsl(var(--accent-foreground));
            transform: scale(1.1);
            backdrop-filter: blur(8px);
        }

        .accordion-arrow {
            transition: all 0.4s cubic-bezier(0.4, 0, 0.2, 1);
            filter: blur(0.5px);
        }

        .accordion-arrow.expanded {
            transform: rotate(180deg);
            filter: blur(0px);
        }

        /* Motion blur animations */
        @keyframes slideDown {
            from {
                opacity: 0;
                transform: translateY(-20px);
                filter: blur(10px);
            }
            to {
                opacity: 1;
                transform: translateY(0);
                filter: blur(0px);
            }
        }

        @keyframes dialogSlideIn {
            from {
                opacity: 0;
                transform: translate(-50%, -48%) scale(0.9);
                filter: blur(20px);
            }
            to {
                opacity: 1;
                transform: translate(-50%, -50%) scale(1);
                filter: blur(0px);
            }
        }

        @keyframes backdropBlurIn {
            from {
                backdrop-filter: blur(0px);
                opacity: 0;
            }
            to {
                backdrop-filter: blur(8px);
                opacity: 1;
            }
        }

        @keyframes gentlePulse {
            0%, 100% {
                transform: scale(1);
                filter: blur(1px);
            }
            50% {
                transform: scale(1.05);
                filter: blur(0.5px);
            }
        }

        @keyframes buttonSlideIn {
            from {
                opacity: 0;
                transform: translateY(-20px) scale(0.8);
                filter: blur(10px);
            }
            to {
                opacity: 1;
                transform: translateY(0) scale(1);
                filter: blur(0px);
            }
        }

        .pdf-category-content.expanded {
            animation: slideDown 0.5s cubic-bezier(0.4, 0, 0.2, 1);
        }

        .pdf-extractor-dialog {
            animation: dialogSlideIn 0.4s cubic-bezier(0.4, 0, 0.2, 1);
        }

        .pdf-extractor-button.show {
            display: flex;
            animation: buttonSlideIn 0.5s cubic-bezier(0.4, 0, 0.2, 1);
        }

        /* Smooth scroll for content */
        .pdf-extractor-content {
            scroll-behavior: smooth;
        }

        .pdf-extractor-content::-webkit-scrollbar {
            width: 6px;
        }

        .pdf-extractor-content::-webkit-scrollbar-track {
            background: hsl(var(--muted));
            border-radius: 3px;
        }

        .pdf-extractor-content::-webkit-scrollbar-thumb {
            background: hsl(var(--primary));
            border-radius: 3px;
            backdrop-filter: blur(4px);
        }

        .pdf-extractor-content::-webkit-scrollbar-thumb:hover {
            backdrop-filter: blur(8px);
        }
    `);

    // Initialize settings
    const defaultSettings = {
        buttonVisible: false, // Hidden by default for non-Lisa apps
        darkMode: window.matchMedia('(prefers-color-scheme: dark)').matches,
        expandedCategories: {}
    };

    function getSettings() {
        const saved = GM_getValue('pdfExtractorSettings');
        return { ...defaultSettings, ...saved };
    }

    function saveSettings(settings) {
        GM_setValue('pdfExtractorSettings', settings);
    }

    // Check if current page is the Lisa app
    function isLisaApp() {
        return window.location.hostname.includes('isu-btech.lisaapp.in') ||
               window.location.hostname.includes('lisaapp.in');
    }

    // Function to check if an element has EXACT classes in EXACT order
    function hasExactClasses(element, classString) {
        const actualClasses = element.className.split(' ').filter(c => c.trim() !== '');
        const expectedClasses = classString.split(' ').filter(c => c.trim() !== '');

        if (actualClasses.length !== expectedClasses.length) {
            return false;
        }

        return actualClasses.every((className, index) => className === expectedClasses[index]);
    }

    // Find Lisa app specific containers with exact class matching
    function findLisaContainers() {
        const containers = [];

        // Define the exact class strings we're looking for
        const parentClassString = "w-full transform overflow-hidden rounded-t-md bg-neutral-50 dark:bg-neutral-900 text-slate-900 dark:text-slate-200 p-6 text-left align-middle shadow-xl transition-all space-y-4 flex flex-col max-w-full h-navScreen opacity-100 translate-y-0";
        const childClassString = "flex justify-between items-center";

        console.log('🔍 Searching for Lisa app containers with exact class matching...');

        // First, find all potential parent elements
        const allElements = document.querySelectorAll('*');

        for (const element of allElements) {
            // Check if this element has the exact parent classes
            if (hasExactClasses(element, parentClassString)) {
                console.log('✅ Found parent container with exact classes');

                // Now look for child elements with exact child classes within this parent
                const children = element.querySelectorAll('*');
                for (const child of children) {
                    if (hasExactClasses(child, childClassString)) {
                        console.log('✅ Found child container with exact classes inside parent');
                        containers.push(child);
                    }
                }

                // Also check if the parent itself contains the child classes as part of its structure
                const parentChildren = element.children;
                for (const child of parentChildren) {
                    if (hasExactClasses(child, childClassString)) {
                        console.log('✅ Found direct child with exact classes');
                        containers.push(child);
                    }
                }
            }
        }

        // Fallback: if no exact hierarchy found, look for elements with the child classes
        if (containers.length === 0) {
            console.log('⚠️ No exact hierarchy found, falling back to child class search');
            for (const element of allElements) {
                if (hasExactClasses(element, childClassString)) {
                    console.log('✅ Found element with child classes (fallback)');
                    containers.push(element);
                }
            }
        }

        console.log(`📦 Found ${containers.length} target containers`);
        return containers;
    }

    function extractPdfLinks() {
        console.log('🔍 Searching for PDF links...');
        const pdfLinks = [];
        const seenUrls = new Set();

        try {
            const allLinks = document.querySelectorAll('a[href]');
            console.log(`Found ${allLinks.length} total links on page`);

            allLinks.forEach(link => {
                try {
                    const href = link.href;
                    if (!href) return;

                    const isPdf = href.toLowerCase().includes('.pdf');
                    const isGView = href.toLowerCase().includes('docs.google.com/gview');
                    const isPdfLike = href.toLowerCase().includes('/pdf') || href.toLowerCase().includes('pdf=true');

                    if ((isPdf || isGView || isPdfLike) && !seenUrls.has(href)) {
                        seenUrls.add(href);
                        const text = link.textContent.trim() || link.title || 'PDF Document';

                        const urlObj = new URL(href);
                        const id = urlObj.searchParams.get('id') ||
                                  urlObj.searchParams.get('resource') ||
                                  href.split('/').pop().split('?')[0].split('.')[0];

                        pdfLinks.push({
                            url: href,
                            text: text,
                            element: link,
                            id: id || 'no-id',
                            domain: urlObj.hostname
                        });
                    }
                } catch (e) {
                    console.log('Error processing link:', e);
                }
            });

            const htmlContent = document.documentElement.outerHTML;
            const pdfUrlPatterns = [
                /https?:\/\/[^\s"']*\.pdf(?:\?[^\s"']*)?/gi,
                /https?:\/\/docs\.google\.com\/gview\?[^\s"']*/gi
            ];

            pdfUrlPatterns.forEach(pattern => {
                let match;
                while ((match = pattern.exec(htmlContent)) !== null) {
                    const url = match[0].replace(/&amp;/g, '&');
                    if (!seenUrls.has(url)) {
                        seenUrls.add(url);
                        const urlObj = new URL(url);
                        const id = urlObj.searchParams.get('id') || urlObj.searchParams.get('resource') || 'no-id';

                        pdfLinks.push({
                            url: url,
                            text: 'PDF found in page content',
                            element: null,
                            id: id,
                            domain: urlObj.hostname
                        });
                    }
                }
            });

        } catch (error) {
            console.error('Error extracting PDF links:', error);
        }

        console.log(`✅ Found ${pdfLinks.length} PDF links total`);
        return pdfLinks;
    }

    function categorizeLinks(pdfLinks) {
        const categories = {};

        pdfLinks.forEach(link => {
            const categoryKey = link.id !== 'no-id' ? link.id : link.domain;
            if (!categories[categoryKey]) {
                categories[categoryKey] = {
                    name: link.id !== 'no-id' ? `ID: ${link.id}` : `Domain: ${link.domain}`,
                    links: [],
                    type: link.id !== 'no-id' ? 'id' : 'domain'
                };
            }
            categories[categoryKey].links.push(link);
        });

        return categories;
    }

    function createPopup(pdfLinks) {
        const existingPopup = document.getElementById('pdf-extractor-popup');
        const existingBackdrop = document.getElementById('pdf-extractor-backdrop');
        if (existingPopup) existingPopup.remove();
        if (existingBackdrop) existingBackdrop.remove();

        const settings = getSettings();

        const backdrop = document.createElement('div');
        backdrop.id = 'pdf-extractor-backdrop';
        backdrop.className = 'pdf-extractor-backdrop';

        const dialog = document.createElement('div');
        dialog.id = 'pdf-extractor-popup';
        dialog.className = 'pdf-extractor-dialog';
        if (settings.darkMode) {
            dialog.classList.add('dark');
        }

        const categories = categorizeLinks(pdfLinks);
        const categoryKeys = Object.keys(categories);

        const header = document.createElement('div');
        header.className = 'pdf-extractor-header';
        header.innerHTML = `
            <h2 class="pdf-extractor-title">📄 PDF Links (${pdfLinks.length})</h2>
            <div class="header-actions">
                <button class="button-toggle" title="Toggle PDF button">
                    ${settings.buttonVisible ? '👁️' : '👁️‍🗨️'}
                </button>
                <button class="theme-toggle" title="Toggle dark mode">
                    ${settings.darkMode ? '☀️' : '🌙'}
                </button>
                <button class="pdf-extractor-close" title="Close">×</button>
            </div>
        `;

        const content = document.createElement('div');
        content.className = 'pdf-extractor-content';

        if (pdfLinks.length === 0) {
            content.innerHTML = `
                <div class="empty-state">
                    <div class="empty-state-icon">📭</div>
                    <h3 style="margin-bottom: 0.5rem;">No PDF Links Found</h3>
                    <p style="margin-bottom: 1rem;">No PDF or Google Docs viewer links were detected.</p>
                    <div class="shortcut-hint">Press Ctrl+Alt+D to try again</div>
                </div>
            `;
        } else {
            const categoriesContainer = document.createElement('div');
            categoriesContainer.className = 'pdf-categories';

            categoryKeys.forEach((key, index) => {
                const category = categories[key];
                const categoryElement = document.createElement('div');
                categoryElement.className = 'pdf-category';

                const isExpanded = settings.expandedCategories[key] !== false;

                categoryElement.innerHTML = `
                    <div class="pdf-category-header">
                        <div class="pdf-category-title">
                            <span>${category.type === 'id' ? '🔑' : '🌐'} ${category.name}</span>
                            <span class="pdf-category-count">${category.links.length}</span>
                        </div>
                        <div class="accordion-arrow ${isExpanded ? 'expanded' : ''}">▼</div>
                    </div>
                    <div class="pdf-category-content ${isExpanded ? 'expanded' : ''}">
                        <div class="pdf-list">
                            ${category.links.map(link => `
                                <div class="pdf-item">
                                    <div class="pdf-icon">📄</div>
                                    <div class="pdf-info">
                                        <div class="pdf-filename">${link.text}</div>
                                        <div class="pdf-url">${link.url.length > 60 ? link.url.substring(0, 60) + '...' : link.url}</div>
                                    </div>
                                    <div class="pdf-actions">
                                        <button class="pdf-action-btn copy-btn">Copy</button>
                                        <button class="pdf-action-btn open-btn">Open</button>
                                    </div>
                                </div>
                            `).join('')}
                        </div>
                    </div>
                `;

                const headerEl = categoryElement.querySelector('.pdf-category-header');
                const contentEl = categoryElement.querySelector('.pdf-category-content');
                const arrowEl = categoryElement.querySelector('.accordion-arrow');

                headerEl.addEventListener('click', () => {
                    const isExpanded = contentEl.classList.contains('expanded');
                    contentEl.classList.toggle('expanded', !isExpanded);
                    arrowEl.classList.toggle('expanded', !isExpanded);

                    settings.expandedCategories[key] = !isExpanded;
                    saveSettings(settings);
                });

                setTimeout(() => {
                    categoryElement.querySelectorAll('.copy-btn').forEach((btn, i) => {
                        btn.addEventListener('click', (e) => {
                            e.stopPropagation();
                            navigator.clipboard.writeText(category.links[i].url);
                            btn.textContent = 'Copied!';
                            setTimeout(() => btn.textContent = 'Copy', 2000);
                        });
                    });

                    categoryElement.querySelectorAll('.open-btn').forEach((btn, i) => {
                        btn.addEventListener('click', (e) => {
                            e.stopPropagation();
                            window.open(category.links[i].url, '_blank');
                        });
                    });

                    categoryElement.querySelectorAll('.pdf-item').forEach((item, i) => {
                        item.addEventListener('click', () => {
                            window.open(category.links[i].url, '_blank');
                        });
                    });
                }, 0);

                categoriesContainer.appendChild(categoryElement);
            });

            content.appendChild(categoriesContainer);
        }

        dialog.appendChild(header);
        dialog.appendChild(content);
        document.body.appendChild(backdrop);
        document.body.appendChild(dialog);

        const themeToggle = dialog.querySelector('.theme-toggle');
        const buttonToggle = dialog.querySelector('.button-toggle');
        const closeBtn = dialog.querySelector('.pdf-extractor-close');

        themeToggle.addEventListener('click', () => {
            settings.darkMode = !settings.darkMode;
            dialog.classList.toggle('dark', settings.darkMode);
            saveSettings(settings);
            themeToggle.innerHTML = settings.darkMode ? '☀️' : '🌙';
        });

        buttonToggle.addEventListener('click', () => {
            settings.buttonVisible = !settings.buttonVisible;
            updateButtonVisibility(settings.buttonVisible);
            saveSettings(settings);
            buttonToggle.innerHTML = settings.buttonVisible ? '👁️' : '👁️‍🗨️';
        });

        const closePopup = () => {
            dialog.remove();
            backdrop.remove();
        };

        closeBtn.addEventListener('click', closePopup);
        backdrop.addEventListener('click', (e) => {
            if (e.target === backdrop) closePopup();
        });

        const keyHandler = (e) => {
            if (e.key === 'Escape') closePopup();
        };
        dialog.addEventListener('keydown', keyHandler);
        closeBtn.focus();
    }

    function showPdfLinks() {
        const pdfLinks = extractPdfLinks();
        createPopup(pdfLinks);
    }

    // Update button visibility based on settings
    function updateButtonVisibility(visible) {
        const isLisa = isLisaApp();

        if (isLisa) {
            // For Lisa app, show/hide the container buttons
            const lisaButtons = document.querySelectorAll('.pdf-extractor-button-lisa');
            lisaButtons.forEach(button => {
                button.style.display = visible ? 'inline-flex' : 'none';
            });
        } else {
            // For non-Lisa apps, show/hide the floating button
            const floatingButton = document.querySelector('.pdf-extractor-button');
            if (floatingButton) {
                if (visible) {
                    floatingButton.classList.add('show');
                } else {
                    floatingButton.classList.remove('show');
                }
            }
        }
    }

    // Add floating button or Lisa app specific buttons
    function addButtons() {
        const existingButtons = document.querySelectorAll('.pdf-extractor-button, .pdf-extractor-button-lisa');
        existingButtons.forEach(btn => btn.remove());

        const settings = getSettings();
        const isLisa = isLisaApp();

        if (isLisa) {
            // Lisa app: Add buttons to specific containers only
            const containers = findLisaContainers();
            console.log(`🎯 Found ${containers.length} Lisa app containers`);

            if (containers.length > 0) {
                containers.forEach((container, index) => {
                    const button = document.createElement('button');
                    button.className = 'pdf-extractor-button-lisa';
                    button.innerHTML = '📄 PDF';
                    button.title = 'Extract PDF links from this page (Ctrl+Alt+D)';
                    button.style.display = settings.buttonVisible ? 'inline-flex' : 'none';

                    button.addEventListener('click', showPdfLinks);

                    container.insertBefore(button, container.firstChild);
                });
            } else {
                addFloatingButton(settings);
            }
        } else {
            addFloatingButton(settings);
        }
    }

    // Add regular floating button (hidden by default)
    function addFloatingButton(settings) {
        const button = document.createElement('button');
        button.className = 'pdf-extractor-button';
        button.innerHTML = '📄 PDF <span class="shortcut-hint">Ctrl+Alt+D</span>';
        button.title = 'Extract PDF links from this page (Ctrl+Alt+D)';

        // Start hidden, will be shown via eye icon in popup
        if (settings.buttonVisible) {
            button.classList.add('show');
        }

        button.addEventListener('click', showPdfLinks);
        document.body.appendChild(button);
    }

    // Keyboard shortcut handler
    function handleKeyPress(e) {
        if (e.ctrlKey && e.altKey && e.keyCode === 68) {
            e.preventDefault();
            e.stopPropagation();
            showPdfLinks();
            return false;
        }
    }

    // Initialize
    function init() {
        const settings = getSettings();
        if (GM_getValue('pdfExtractorSettings') === undefined) {
            settings.darkMode = window.matchMedia('(prefers-color-scheme: dark)').matches;
            saveSettings(settings);
        }

        window.addEventListener('keydown', handleKeyPress, true);

        // Add main buttons
        setTimeout(addButtons, 1000);

        console.log('✅ PDF Extractor with Motion Blur ready!');
        console.log('🌐 Current site:', window.location.hostname);
        console.log('📱 Lisa app detected:', isLisaApp());
        console.log('👁️ Button visible:', settings.buttonVisible);
    }

    let lastUrl = location.href;
    const observer = new MutationObserver(() => {
        const url = location.href;
        if (url !== lastUrl) {
            lastUrl = url;
            setTimeout(init, 500);
        }

        if (isLisaApp()) {
            setTimeout(() => {
                const containers = findLisaContainers();
                const existingButtons = document.querySelectorAll('.pdf-extractor-button-lisa');
                if (containers.length > existingButtons.length) {
                    addButtons();
                }
            }, 300);
        }
    });

    if (document.readyState === 'loading') {
        document.addEventListener('DOMContentLoaded', () => {
            init();
            observer.observe(document, { subtree: true, childList: true });
        });
    } else {
        init();
        observer.observe(document, { subtree: true, childList: true });
    }

})();

```
