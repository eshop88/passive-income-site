# passive-income-site

passives-einkommen/
├── _posts/                  # Automatisch generierte Produktreviews
├── _includes/               # Jekyll Templates
├── _layouts/                # Jekyll Layouts
├── assets/                  # CSS/JS/Images
├── scripts/                 # Python Automation
│   ├── generate_content.py  # Content Generation
│   ├── seo_optimizer.py     # SEO Tools
│   └── deploy.py            # Deployment Script
├── .github/workflows/       # GitHub Actions
│   └── daily_update.yml     # Tägliche Automation
├── _config.yml              # Jekyll Config
└── CNAME                    # Custom Domain (optional)
title: Home & Kitchen Reviews
description: Expert reviews of the best home and kitchen products
url: "https://yourusername.github.io" # Oder Custom Domain
amazon_tag: YOUR_AMAZON_AFFILIATE_ID
theme: minima
plugins:
  - jekyll-seo-tag
  - jekyll-sitemap
defaults:
  - scope:
      path: ""
      type: "posts"
    values:
      layout: "post"
      comments: true
      share: true
import openai
import requests
import yaml
from datetime import datetime

# Konfiguration
openai.api_key = "MEIN_OPENAI_API_KEY"
AMAZON_API_KEY = "MEIN_AMAZON_API_KEY"
AMAZON_AFFILIATE_ID = "MEIN_AMAZON_AFFILIATE_ID"

def get_amazon_product(category="Home & Kitchen"):
    # Platzhalter für Amazon Product Advertising API
    params = {
        "Operation": "ItemSearch",
        "SearchIndex": category,
        "ResponseGroup": "ItemAttributes,Offers,Images",
        "Sort": "salesrank"
    }
    # Mock Response - im echten System durch API Call ersetzen
    return {
        "title": "Premium Kitchen Knife Set",
        "asin": "B0ABCD1234",
        "price": "$89.99",
        "image_url": "https://m.media-amazon.com/images/I/71abcd1234._AC_SL1500_.jpg",
        "features": ["8-piece set", "Ergonomic handles", "Stainless steel"],
        "rating": 4.7
    }

def generate_review(product):
    prompt = f"""Write a detailed, SEO-optimized 1500-word product review for {product['title']} with:
    - Engaging introduction
    - Key features analysis
    - Pros and cons
    - Comparison to competitors
    - Verdict with affiliate link
    
    Product details: {product}"""
    
    response = openai.ChatCompletion.create(
        model="gpt-3.5-turbo",
        messages=[{"role": "user", "content": prompt}],
        temperature=0.7
    )
    return response.choices[0].message.content

def create_jekyll_post(content, product):
    today = datetime.now().strftime("%Y-%m-%d")
    filename = f"_posts/{today}-{product['title'].lower().replace(' ', '-')}.md"
    
    front_matter = {
        "layout": "post",
        "title": f"Review: {product['title']}",
        "date": datetime.now().strftime("%Y-%m-%d %H:%M:%S %z"),
        "categories": ["review"],
        "product": product,
        "affiliate_link": f"https://www.amazon.com/dp/{product['asin']}/?tag={AMAZON_AFFILIATE_ID}"
    }
    
    with open(filename, 'w') as f:
        f.write("---\n")
        f.write(yaml.dump(front_matter))
        f.write("---\n\n")
        f.write(content)

if __name__ == "__main__":
    product = get_amazon_product()
    review = generate_review(product)
    create_jekyll_post(review, product)
name: Daily Content Update

on:
  schedule:
    - cron: '0 12 * * *' # Täglich um 12:00 UTC
  workflow_dispatch:

jobs:
  generate-content:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.9'
      
      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install openai pyyaml requests
      
      - name: Generate content
        env:
          OPENAI_API_KEY: ${{ secrets.OPENAI_API_KEY }}
          AMAZON_API_KEY: ${{ secrets.AMAZON_API_KEY }}
          AMAZON_AFFILIATE_ID: ${{ secrets.AMAZON_AFFILIATE_ID }}
        run: |
          python scripts/generate_content.py
      
      - name: Commit and push changes
        run: |
          git config --global user.name "GitHub Actions"
          git config --global user.email "actions@github.com"
          git add _posts/
          git commit -m "Automatic content update"
          git push
from bs4 import BeautifulSoup
import os
import re

def optimize_seo(filepath):
    with open(filepath, 'r+') as f:
        content = f.read()
        soup = BeautifulSoup(content, 'html.parser')
        
        # Meta Description hinzufügen
        if not soup.find('meta', attrs={'name': 'description'}):
            meta = soup.new_tag('meta', attrs={'name': 'description'})
            first_paragraph = soup.find('p').get_text()[:160]
            meta['content'] = first_paragraph
            soup.head.append(meta)
        
        # Heading Struktur optimieren
        for h in soup.find_all(re.compile('^h[1-6]$')):
            if h.name == 'h1' and not 'review' in h.get_text().lower():
                h.string = "Review: " + h.get_text()
        
        # Affiliate Links tracken
        for a in soup.find_all('a', href=re.compile('amazon.com')):
            if 'tag=' not in a['href']:
                a['href'] = a['href'] + f"?tag={AMAZON_AFFILIATE_ID}"
        
        f.seek(0)
        f.write(str(soup))
        f.truncate()

if __name__ == "__main__":
    for root, _, files in os.walk('_posts'):
        for file in files:
            if file.endswith('.html'):
                optimize_seo(os.path.join(root, file))
---
layout: post
title: "Review: Premium Kitchen Knife Set"
date: 2023-01-01 12:00:00 -0500
categories: review
product:
  title: "Premium Kitchen Knife Set"
  asin: "B0ABCD1234"
  price: "$89.99"
  image_url: "https://m.media-amazon.com/images/I/71abcd1234._AC_SL1500_.jpg"
  features: ["8-piece set", "Ergonomic handles", "Stainless steel"]
  rating: 4.7
affiliate_link: "https://www.amazon.com/dp/B0ABCD1234/?tag=YOUR_AFFILIATE_ID"
---

[Inhalte werden automatisch generiert...]
<script>
// Basic Conversion Tracking
document.querySelectorAll('a[href*="amazon.com"]').forEach(link => {
    link.addEventListener('click', function() {
        fetch(`https://your-tracking-endpoint.com/track?url=${encodeURIComponent(this.href)}`, {
            method: 'GET',
            mode: 'no-cors'
        });
    });
});
</script>

_config.yml
title: Home & Kitchen Reviews
description: Automatisierte Produktbewertungen
url: https://DEIN_NAME.github.io/passives-einkommen
amazon_tag: DEIN_AMAZON_AFFILIATE_ID
theme: minima
