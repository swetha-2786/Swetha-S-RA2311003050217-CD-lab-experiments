def parse_grammar(text):
    grammar, order = {}, []
    for line in text.strip().split('\n'):
        line = line.strip()
        if not line or '->' not in line:
            continue
        lhs, rhs = line.split('->')[0].strip(), line.split('->')[1].strip()
        grammar[lhs] = [alt.strip().split() for alt in rhs.split('|')]
        if lhs not in order:
            order.append(lhs)
    return grammar, order


def format_grammar(grammar, order=None):
    if order is None:
        order = list(grammar.keys())
    lines = []
    for nt in order:
        if nt in grammar:
            lines.append(f"  {nt} -> {' | '.join(' '.join(a) for a in grammar[nt])}")
    return '\n'.join(lines)


def eliminate_immediate_lr(nt, prods):
    recursive, non_recursive = [], []
    for p in prods:
        if p[0] == nt:
            recursive.append(p[1:] if len(p) > 1 else ['e'])
        else:
            non_recursive.append(p)

    if not recursive:
        return {nt: prods}, []

    new_nt = nt + "'"
    new_prods = []
    for p in non_recursive:
        new_prods.append(([new_nt] if p == ['e'] else p + [new_nt]))
    if not new_prods:
        new_prods.append([new_nt])

    new_nt_prods = []
    for p in recursive:
        new_nt_prods.append(([new_nt] if p == ['e'] else p + [new_nt]))
    new_nt_prods.append(['e'])

    return {nt: new_prods, new_nt: new_nt_prods}, [new_nt]


def eliminate_left_recursion(grammar, order):
    g = {nt: [list(p) for p in grammar[nt]] for nt in grammar}
    new_order = list(order)
    nts = list(order)

    for i, ai in enumerate(nts):
        for j in range(i):
            aj = nts[j]
            updated = []
            for p in g.get(ai, []):
                if p[0] == aj:
                    suffix = p[1:] if len(p) > 1 else []
                    for ap in g.get(aj, []):
                        updated.append((suffix if ap == ['e'] else ap + suffix) or ['e'])
                else:
                    updated.append(p)
            g[ai] = updated

        result, new_nts = eliminate_immediate_lr(ai, g.get(ai, []))
        g.update(result)
        for nnt in new_nts:
            if nnt not in new_order:
                new_order.insert(new_order.index(ai) + 1, nnt)
    return g, new_order


def left_factoring(grammar, order):
    g = {nt: [list(p) for p in grammar[nt]] for nt in grammar}
    new_order = list(order)
    changed = True

    while changed:
        changed = False
        temp_g, temp_order = {}, list(new_order)

        for nt in list(new_order):
            if nt not in g:
                continue
            prods = g[nt]
            groups = {}
            for p in prods:
                groups.setdefault(p[0] if p else '', []).append(p)

            needs = any(len(ps) > 1 and k != 'e' for k, ps in groups.items())
            if not needs:
                temp_g[nt] = prods
                continue

            changed = True
            new_prods = []
            for prefix, ps in groups.items():
                if len(ps) <= 1 or prefix == 'e':
                    new_prods.extend(ps)
                    continue
                cp = list(ps[0])
                for p in ps[1:]:
                    np = []
                    for k in range(min(len(cp), len(p))):
                        if cp[k] == p[k]:
                            np.append(cp[k])
                        else:
                            break
                    cp = np
                if not cp:
                    new_prods.extend(ps)
                    continue
                nnt = nt + "'"
                while nnt in g or nnt in temp_g:
                    nnt += "'"
                new_prods.append(cp + [nnt])
                temp_g[nnt] = [p[len(cp):] if p[len(cp):] else ['e'] for p in ps]
                temp_order.insert(temp_order.index(nt) + 1, nnt)
            temp_g[nt] = new_prods

        g, new_order = temp_g, temp_order
    return g, new_order


def main():
    print("Elimination of Left Recursion and Left Factoring")
    print("-" * 50)

    print("\n--- Example 1: Direct Left Recursion ---")
    g1, o1 = parse_grammar("E -> E + T | T\nT -> T * F | F\nF -> ( E ) | id")
    print("Original:\n" + format_grammar(g1, o1))
    r1, ro1 = eliminate_left_recursion(g1, o1)
    print("After elimination:\n" + format_grammar(r1, ro1))

    print("\n--- Example 2: Indirect Left Recursion ---")
    g2, o2 = parse_grammar("S -> A a | b\nA -> S d | e")
    print("Original:\n" + format_grammar(g2, o2))
    r2, ro2 = eliminate_left_recursion(g2, o2)
    print("After elimination:\n" + format_grammar(r2, ro2))

    print("\n--- Example 3: Left Factoring (Dangling Else) ---")
    g3, o3 = parse_grammar("S -> i E t S e S | i E t S | a\nE -> b")
    print("Original:\n" + format_grammar(g3, o3))
    r3, ro3 = left_factoring(g3, o3)
    print("After factoring:\n" + format_grammar(r3, ro3))

    print("\n--- Example 4: Left Factoring ---")
    g4, o4 = parse_grammar("A -> a b B | a b c | a d\nB -> c | d")
    print("Original:\n" + format_grammar(g4, o4))
    r4, ro4 = left_factoring(g4, o4)
    print("After factoring:\n" + format_grammar(r4, ro4))

    print("\n--- Example 5: Combined ---")
    g5, o5 = parse_grammar("E -> E + T | E - T | T\nT -> T * F | T / F | F\nF -> ( E ) | id | num")
    print("Original:\n" + format_grammar(g5, o5))
    r5, ro5 = eliminate_left_recursion(g5, o5)
    print("After LR elimination:\n" + format_grammar(r5, ro5))

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
        g, o = parse_grammar('\n'.join(lines))
        print("Original:\n" + format_grammar(g, o))
        choice = input("1=LR Elimination, 2=Left Factoring, 3=Both: ").strip()
        if choice == '1':
            r, ro = eliminate_left_recursion(g, o)
            print("Result:\n" + format_grammar(r, ro))
        elif choice == '2':
            r, ro = left_factoring(g, o)
            print("Result:\n" + format_grammar(r, ro))
        elif choice == '3':
            r, ro = eliminate_left_recursion(g, o)
            print("After LR:\n" + format_grammar(r, ro))
            r, ro = left_factoring(r, ro)
            print("After LF:\n" + format_grammar(r, ro))


if __name__ == "__main__":
    main()
