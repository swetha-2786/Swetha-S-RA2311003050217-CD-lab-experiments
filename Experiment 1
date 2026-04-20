import re

KEYWORDS = {
    'if', 'else', 'while', 'for', 'do', 'break', 'continue', 'return',
    'int', 'float', 'char', 'double', 'void', 'main', 'include', 'printf',
    'scanf', 'struct', 'switch', 'case', 'default', 'typedef', 'enum',
    'static', 'const', 'sizeof', 'long', 'short', 'unsigned', 'signed'
}

TOKEN_PATTERNS = [
    ('COMMENT_ML',  r'/\*[\s\S]*?\*/'),
    ('COMMENT_SL',  r'//.*'),
    ('HEADER',      r'<[a-zA-Z_]\w*\.h>'),
    ('STRING',      r'"[^"]*"'),
    ('CHAR_LIT',    r"'[^']*'"),
    ('FLOAT_NUM',   r'\d+\.\d+'),
    ('INTEGER',     r'\d+'),
    ('PREPROCESSOR', r'#\w+'),
    ('IDENTIFIER',  r'[a-zA-Z_]\w*'),
    ('OP_DOUBLE',   r'==|!=|<=|>=|&&|\|\||<<|>>|\+\+|--|[+\-*/]=|%='),
    ('OP_SINGLE',   r'[+\-*/%=<>!&|^~]'),
    ('SPECIAL',     r'[(){}\[\];,.:?#]'),
    ('NEWLINE',     r'\n'),
    ('WHITESPACE',  r'[ \t]+'),
    ('UNKNOWN',     r'.'),
]

master_re = '|'.join(f'(?P<{name}>{pat})' for name, pat in TOKEN_PATTERNS)
compiled_re = re.compile(master_re)


def lexical_analyze(code):
    tokens = []
    for m in compiled_re.finditer(code):
        ttype = m.lastgroup
        tval = m.group()

        if ttype in ('WHITESPACE', 'NEWLINE'):
            continue
        if ttype in ('COMMENT_ML', 'COMMENT_SL'):
            tokens.append(('COMMENT', tval))
            continue
        if ttype == 'IDENTIFIER':
            ttype = 'KEYWORD' if tval in KEYWORDS else 'IDENTIFIER'
        if ttype in ('OP_DOUBLE', 'OP_SINGLE'):
            ttype = 'OPERATOR'
        if ttype == 'SPECIAL':
            ttype = 'SPECIAL_SYMBOL'

        tokens.append((ttype, tval))
    return tokens


def display_tokens(tokens):
    print(f"\n{'Token Type':<20} {'Token Value':<40}")
    print("-" * 60)
    for ttype, tval in tokens:
        print(f"{ttype:<20} {tval.replace(chr(10), '\\n'):<40}")
    print("-" * 60)

    counts = {}
    for ttype, _ in tokens:
        counts[ttype] = counts.get(ttype, 0) + 1
    print("\nSummary:")
    for t, c in sorted(counts.items()):
        print(f"  {t:<20}: {c}")
    print(f"  {'TOTAL':<20}: {len(tokens)}")


def main():
    print("LEXICAL ANALYZER")
    print("-" * 40)

    sample = '''
#include <stdio.h>

int main() {
    int a = 10;
    float b = 20.5;
    char c = 'x';

    // single line comment
    /* multi line
       comment */

    if (a >= 10 && b != 0) {
        printf("Hello, World!");
        a++;
    } else {
        a = a + b;
    }

    for (int i = 0; i < 10; i++) {
        a += i;
    }

    return 0;
}
'''

    print("\nSource Code:")
    print(sample)

    tokens = lexical_analyze(sample)
    display_tokens(tokens)

    print("\nEnter your own code (type 'exit' to quit, blank line to process):")
    while True:
        lines = []
        print("\n>>> ", end="")
        while True:
            try:
                line = input()
            except EOFError:
                return
            if line.strip().lower() == 'exit':
                return
            if line == "" and lines:
                break
            lines.append(line)
        if lines:
            display_tokens(lexical_analyze('\n'.join(lines)))


if __name__ == "__main__":
    main()
