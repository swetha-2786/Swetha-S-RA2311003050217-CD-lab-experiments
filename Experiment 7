def parse_grammar(text):
    grammar, productions, order = {}, [], []
    for line in text.strip().split('\n'):
        line = line.strip()
        if not line or '->' not in line:
            continue
        lhs = line.split('->')[0].strip()
        rhs = line.split('->')[1].strip()
        alts = [alt.strip().split() for alt in rhs.split('|')]
        if lhs not in grammar:
            grammar[lhs] = []
            order.append(lhs)
        for alt in alts:
            grammar[lhs].append(alt)
            productions.append((lhs, alt))
    return grammar, productions, order


def shift_reduce_parse(inp, grammar, productions, start):
    tokens = inp.strip().split() + ['$']
    stack = ['$']
    ptr, step = 0, 0

    print(f"\nParsing: {inp}")
    print(f"{'Step':<6}{'Stack':<30}{'Input':<25}{'Action':<25}")
    print("-" * 85)

    max_steps = 100
    while step < max_steps:
        step += 1
        sstr = ' '.join(stack)
        rem = ' '.join(tokens[ptr:])

        if len(stack) == 2 and stack[1] == start and tokens[ptr] == '$':
            print(f"{step:<6}{sstr:<30}{rem:<25}{'ACCEPT':<25}")
            return True

        reduced = False
        for lhs, rhs in productions:
            rlen = len(rhs)
            if rlen <= len(stack) - 1 and stack[-rlen:] == rhs:
                action = f"REDUCE {lhs} -> {' '.join(rhs)}"
                print(f"{step:<6}{sstr:<30}{rem:<25}{action:<25}")
                for _ in range(rlen):
                    stack.pop()
                stack.append(lhs)
                reduced = True
                break

        if reduced:
            continue

        if ptr < len(tokens) and tokens[ptr] != '$':
            action = f"SHIFT {tokens[ptr]}"
            print(f"{step:<6}{sstr:<30}{rem:<25}{action:<25}")
            stack.append(tokens[ptr])
            ptr += 1
        elif tokens[ptr] == '$' and len(stack) > 1:
            did_reduce = False
            for lhs, rhs in productions:
                rlen = len(rhs)
                if rlen <= len(stack) - 1 and stack[-rlen:] == rhs:
                    action = f"REDUCE {lhs} -> {' '.join(rhs)}"
                    print(f"{step:<6}{sstr:<30}{rem:<25}{action:<25}")
                    for _ in range(rlen):
                        stack.pop()
                    stack.append(lhs)
                    did_reduce = True
                    break
            if not did_reduce:
                print(f"{step:<6}{sstr:<30}{rem:<25}{'ERROR':<25}")
                return False
        else:
            print(f"{step:<6}{sstr:<30}{rem:<25}{'ERROR':<25}")
            return False

    print("ERROR: max steps exceeded")
    return False


def main():
    print("Shift-Reduce Parser")
    print("-" * 40)

    print("\n--- Example 1: Arithmetic Expression ---")
    g1_text = "E -> E + T\nE -> T\nT -> T * F\nT -> F\nF -> ( E )\nF -> id"
    g1, p1, o1 = parse_grammar(g1_text)
    print("Grammar:")
    for i, (lhs, rhs) in enumerate(p1):
        print(f"  ({i+1}) {lhs} -> {' '.join(rhs)}")

    for s in ["id + id * id", "id * id + id", "( id + id ) * id"]:
        shift_reduce_parse(s, g1, p1, o1[0])

    print("\n--- Example 2: Simple Grammar ---")
    g2_text = "S -> a A B b\nA -> c\nB -> d"
    g2, p2, o2 = parse_grammar(g2_text)
    print("Grammar:")
    for i, (lhs, rhs) in enumerate(p2):
        print(f"  ({i+1}) {lhs} -> {' '.join(rhs)}")
    shift_reduce_parse("a c d b", g2, p2, o2[0])

    print("\n--- Example 3: Ambiguous Grammar ---")
    g3_text = "E -> E + E\nE -> E * E\nE -> id"
    g3, p3, o3 = parse_grammar(g3_text)
    print("Grammar:")
    for i, (lhs, rhs) in enumerate(p3):
        print(f"  ({i+1}) {lhs} -> {' '.join(rhs)}")
    shift_reduce_parse("id + id * id", g3, p3, o3[0])

    print("\nEnter grammar (blank line to process, 'exit' to quit):")
    while True:
        lines = []
        try:
            while True:
                l = input("  ")
                if l.strip().lower() == 'exit':
                    return
                if l.strip() == '':
                    break
                lines.append(l)
        except EOFError:
            return
        if not lines:
            continue
        g, p, o = parse_grammar('\n'.join(lines))
        print("Grammar:")
        for i, (lhs, rhs) in enumerate(p):
            print(f"  ({i+1}) {lhs} -> {' '.join(rhs)}")
        while True:
            try:
                s = input("String to parse (or 'back'): ").strip()
            except EOFError:
                return
            if s.lower() == 'back':
                break
            if s:
                shift_reduce_parse(s, g, p, o[0])


if __name__ == "__main__":
    main()
