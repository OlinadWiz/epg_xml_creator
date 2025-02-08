import os
import gzip
import xml.sax
import requests
from pathlib import Path
from xml.sax import ContentHandler
from typing import Set, Dict, Any

# Lista degli URL da processare
urls = [
    'https://www.dropbox.com/scl/fi/7r7h1jdufwoplnhhxkism/m3u4u-103216-593044-EPG.xml?rlkey=606vswc00na76l51otnz116ed&st=q273qocn&dl=1',
    'https://www.dropbox.com/scl/fi/tsj8796ea6krin4pv4t32/m3u4u-103216-595541-EPG.xml?rlkey=tu42144366j5w0n2s8fc1ogvp&st=2gg7ylx2&dl=1',
    'https://epgshare01.online/epgshare01/epg_ripper_US1.xml.gz',
    'https://epgshare01.online/epgshare01/epg_ripper_US_LOCALS2.xml.gz',
    'https://epgshare01.online/epgshare01/epg_ripper_CA1.xml.gz',
    'https://epgshare01.online/epgshare01/epg_ripper_UK1.xml.gz',
    'https://epgshare01.online/epgshare01/epg_ripper_AU1.xml.gz',
    'https://epgshare01.online/epgshare01/epg_ripper_IE1.xml.gz',
    'https://epgshare01.online/epgshare01/epg_ripper_DE1.xml.gz',
    'https://epgshare01.online/epgshare01/epg_ripper_ZA1.xml.gz',
    'https://epgshare01.online/epgshare01/epg_ripper_FR1.xml.gz',
    'https://epgshare01.online/epgshare01/epg_ripper_CL1.xml.gz',
    'https://epgshare01.online/epgshare01/epg_ripper_BR1.xml.gz',
    'https://epgshare01.online/epgshare01/epg_ripper_BG1.xml.gz',
    'https://epgshare01.online/epgshare01/epg_ripper_DK1.xml.gz',
    'https://epgshare01.online/epgshare01/epg_ripper_GR1.xml.gz',
    'https://epgshare01.online/epgshare01/epg_ripper_IL1.xml.gz',
    'https://epgshare01.online/epgshare01/epg_ripper_IT1.xml.gz',
    'https://epgshare01.online/epgshare01/epg_ripper_MY1.xml.gz',
    'https://epgshare01.online/epgshare01/epg_ripper_MX1.xml.gz',
    'https://epgshare01.online/epgshare01/epg_ripper_NL1.xml.gz',
    'https://epgshare01.online/epgshare01/epg_ripper_NZ1.xml.gz',
    'https://epgshare01.online/epgshare01/epg_ripper_CZ1.xml.gz',
    'https://epgshare01.online/epgshare01/epg_ripper_SG1.xml.gz',
    'https://epgshare01.online/epgshare01/epg_ripper_PK1.xml.gz',
    'https://epgshare01.online/epgshare01/epg_ripper_RO1.xml.gz',
    'https://epgshare01.online/epgshare01/epg_ripper_CH1.xml.gz',
    'https://epgshare01.online/epgshare01/epg_ripper_PL1.xml.gz',
    'https://epgshare01.online/epgshare01/epg_ripper_SE1.xml.gz',
    'https://epgshare01.online/epgshare01/epg_ripper_UY1.xml.gz',
    'https://epgshare01.online/epgshare01/epg_ripper_CO1.xml.gz',
    'https://epgshare01.online/epgshare01/epg_ripper_PT1.xml.gz',
    'https://epgshare01.online/epgshare01/epg_ripper_ES1.xml.gz',
    'https://epgshare01.online/epgshare01/epg_ripper_TR1.xml.gz',
    'https://epgshare01.online/epgshare01/epg_ripper_FANDUEL1.xml.gz',
    'https://epg.pw/api/epg.xml?channel_id=8486',
    'https://epg.pw/api/epg.xml?channel_id=12358',
    'https://epg.pw/api/epg.xml?channel_id=9206',
]

# Percorsi dei file
class EPGContentHandler(ContentHandler):
    def __init__(self, valid_tvg_ids: Set[str], output_file):
        super().__init__()
        self.valid_tvg_ids = valid_tvg_ids
        self.output_file = output_file
        self.current_tag = None
        self.current_attrs: Dict[str, str] = {}
        self.captured_xml = []
        self.inside_target_element = False

    def escape_attr(self, text: str) -> str:
        return text.replace('&', '&amp;').replace('"', '&quot;') \
            .replace('<', '&lt;').replace('>', '&gt;')

    def startElement(self, name: str, attrs: Dict[str, str]):
        tag_name = name.lower()

        if tag_name in ('channel', 'programme'):
            self.current_tag = tag_name
            self.current_attrs = dict(attrs)
            self.captured_xml = [f'<{name}']

            for k, v in attrs.items():
                self.captured_xml.append(f' {k}="{self.escape_attr(v)}"')
            self.captured_xml.append('>')
            self.inside_target_element = True
        elif self.inside_target_element:
            self.captured_xml.append(f'<{name}')
            for k, v in attrs.items():
                self.captured_xml.append(f' {k}="{self.escape_attr(v)}"')
            self.captured_xml.append('>')

    def characters(self, content: str):
        if self.inside_target_element:
            escaped = content.replace('&', '&amp;').replace('<', '&lt;').replace('>', '&gt;')
            self.captured_xml.append(escaped)

    def endElement(self, name: str):
        tag_name = name.lower()

        if tag_name in ('channel', 'programme') and self.inside_target_element:
            self.captured_xml.append(f'</{name}>')

            tvg_attr = None
            if tag_name == 'channel':
                tvg_attr = self.current_attrs.get('id') or self.current_attrs.get('ID')
            else:
                tvg_attr = self.current_attrs.get('channel') or self.current_attrs.get('CHANNEL')

            if tvg_attr and tvg_attr in self.valid_tvg_ids:
                self.output_file.write(''.join(self.captured_xml) + '\n')

            self.inside_target_element = False
            self.current_tag = None
            self.current_attrs = {}
            self.captured_xml = []
        elif self.inside_target_element:
            self.captured_xml.append(f'</{name}>')


def process_epg_url(url: str, valid_ids: Set[str], output_file):  # Rimosso async
    print(f"Elaborazione: {url}")

    try:
        response = requests.get(url, stream=True)
        response.raise_for_status()
    except requests.RequestException as e:
        print(f"Errore di rete per {url}: {str(e)}")
        return

    try:
        content_stream = response.raw
        if url.endswith('.gz'):
            content_stream = gzip.GzipFile(fileobj=content_stream)

        parser = xml.sax.make_parser()
        handler = EPGContentHandler(valid_ids, output_file)
        parser.setContentHandler(handler)
        parser.parse(content_stream)
    except Exception as e:
        print(f"Errore durante il parsing di {url}: {str(e)}")


def main():
    script_dir = Path(__file__).parent
    tvg_ids_file = script_dir / 'tvg-ids.txt'
    output_file = script_dir / 'epg.xml'

    try:
        with open(tvg_ids_file, 'r', encoding='utf-8') as f:
            valid_ids = {line.strip() for line in f if line.strip()}
    except IOError as e:
        print(f"Errore durante la lettura di {tvg_ids_file}: {str(e)}")
        return 1

    with open(output_file, 'w', encoding='utf-8') as out:
        out.write('<?xml version="1.0" encoding="UTF-8"?>\n<tv>\n')

        for url in urls:
            try:
                process_epg_url(url, valid_ids, out)  # Rimossa la keyword await
            except Exception as e:
                print(f"Errore durante l'elaborazione di {url}: {str(e)}")

        out.write('</tv>\n')

    print(f"\nCompletato! EPG filtrato scritto in => {output_file}")


if __name__ == '__main__':
    try:
        main()
    except Exception as e:
        print(f"Errore fatale: {str(e)}")
        exit(1)
