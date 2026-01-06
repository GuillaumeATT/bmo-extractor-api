from flask import Flask, request, jsonify
import requests
from bs4 import BeautifulSoup

app = Flask(__name__)

@app.route("/extract")
def extract():
    url = request.args.get("url")
    if not url:
        return jsonify({"error": "Missing url parameter"}), 400

    try:
        r = requests.get(url, timeout=10, headers={
            "User-Agent": "Mozilla/5.0"
        })
        r.raise_for_status()
    except Exception as e:
        return jsonify({"error": str(e)}), 500

    soup = BeautifulSoup(r.text, "html.parser")

    # title
    title = soup.title.string if soup.title else ""

    # headings
    headings = []
    for level in range(1, 4):
        for h in soup.find_all(f"h{level}"):
            headings.append({
                "level": level,
                "text": h.get_text(strip=True)
            })

    # main text (simple version)
    paragraphs = [p.get_text(" ", strip=True) for p in soup.find_all("p")]
    main_text = "\n\n".join(paragraphs[:50])

    return jsonify({
        "final_url": url,
        "title": title,
        "main_text": main_text,
        "headings": headings
    })

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=10000)
