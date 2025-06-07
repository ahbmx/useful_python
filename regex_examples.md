import re

wwn_pattern = re.compile(r'^(?:.*\b)?([0-9a-fA-F]{2}(?::[0-9a-fA-F]{2}){7})\b')
