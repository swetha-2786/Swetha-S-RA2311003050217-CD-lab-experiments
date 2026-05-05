def parse_grammar(text):
    grammar, order = {}, []
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
        grammar[lhs].extend(alts)
    return grammar, order


def get_nts(grammar):
    return set(grammar.keys())


def get_terminals(grammar):
    nts = get_nts(grammar)
    terms = set()
    for nt in grammar:
        for prod in grammar[nt]:
            for sym in prod:
                if sym not in nts and sym != 'e':
                    terms.add(sym)
    return terms


def compute_leading(grammar, order):
    nts = get_nts(grammar)
    leading = {nt: set() for nt in nts}
    changed = True

    while changed:
        changed = False
        for nt in order:
            for prod in grammar[nt]:
                if prod == ['e']:
                    continue
                for i, sym in enumerate(prod):
                    if sym not in nts:
                        if sym not in leading[nt]:
                            leading[nt].add(sym)
                            changed = True
                        break
                    else:
                        new = leading[sym] - leading[nt]
                        if new:
                            leading[nt].update(new)
                            changed = True
                        if i + 1 < len(prod):
                            nxt = prod[i + 1]
                            if nxt not in nts and nxt not in leading[nt]:
                                leading[nt].add(nxt)
                                changed = True
                        break
    return leading


def compute_trailing(grammar, order):
    nts = get_nts(grammar)
    trailing = {nt: set() for nt in nts}
    changed = True

    while changed:
        changed = False
        for nt in order:
            for prod in grammar[nt]:
                if prod == ['e']:
                    continue
                for i in range(len(prod) - 1, -1, -1):
                    sym = prod[i]
                    if sym not in nts:
                        if sym not in trailing[nt]:
                            trailing[nt].add(sym)
                            changed = True
                        break
                    else:
                        new = trailing[sym] - trailing[nt]
                        if new:
                            trailing[nt].update(new)
                            changed = True
                        if i - 1 >= 0:
                            prev = prod[i - 1]
                            if prev not in nts and prev not in trailing[nt]:
                                trailing[nt].add(prev)
                                changed = True
                        break
    return trailing


def show_sets(order, leading, trailing):
    print(f"\n{'Non-Terminal':<15} {'LEADING':<30} {'TRAILING':<30}")
    print("-" * 70)
    for nt in order:
        l_str = "{ " + ", ".join(sorted(leading[nt])) + " }"
        t_str = "{ " + ", ".join(sorted(trailing[nt])) + " }"
        print(f"  {nt:<13} {l_str:<30} {t_str:<30}")


def build_precedence_table(grammar, order, leading, trailing):
    nts = get_nts(grammar)
    terminals = get_terminals(grammar)
    all_terms = terminals | {'$'}

    prec = {t1: {t2: ' ' for t2 in all_terms} for t1 in all_terms}

    for nt in order:
        for prod in grammar[nt]:
            if prod == ['e']:
                continue
            for i in range(len(prod)):
                cur = prod[i]
                if i + 1 < len(prod):
                    nxt = prod[i + 1]
                    if cur not in nts and nxt not in nts:
                        prec[cur][nxt] = '='
                    if cur not in nts and nxt in nts:
                        for t in leading[nxt]:
                            prec[cur][t] = '<'
                    if cur in nts and nxt not in nts:
                        for t in trailing[cur]:
                            prec[t][nxt] = '>'
                    if cur in nts and nxt in nts:
                        for t1 in trailing[cur]:
                            for t2 in leading[nxt]:
                                prec[t1][t2] = '='

                if i + 2 < len(prod):
                    n1, n2 = prod[i + 1], prod[i + 2]
                    if cur not in nts and n1 in nts and n2 not in nts:
                        if prec[cur][n2] == ' ':
                            prec[cur][n2] = '='

    start = order[0]
    for t in leading.get(start, set()):
        prec['$'][t] = '<'
    for t in trailing.get(start, set()):
        prec[t]['$'] = '>'

    return prec, all_terms


def show_precedence(prec, terminals):
    tsorted = sorted(terminals - {'$'}) + ['$']
    w = 6
    print(f"\nOperator Precedence Table:")
    header = f"{'':>8}" + "".join(f"{t:>{w}}" for t in tsorted)
    print(header)
    print("-" * len(header))
    for t1 in tsorted:
        row = f"{t1:>8}"
        for t2 in tsorted:
            row += f"{prec.get(t1, {}).get(t2, ' '):>{w}}"
        print(row)
    print("\n< (less than), = (equal), > (greater than)")


def main():
    print("Computation of LEADING and TRAILING Sets")
    print("-" * 45)

    print("\n--- Example 1: Expression Grammar ---")
    g1, o1 = parse_grammar("E -> E + T | T\nT -> T * F | F\nF -> ( E ) | id")
    print("Grammar:")
    for nt in o1:
        print(f"  {nt} -> {' | '.join(' '.join(p) for p in g1[nt])}")
    l1 = compute_leading(g1, o1)
    t1 = compute_trailing(g1, o1)
    show_sets(o1, l1, t1)
    p1, terms1 = build_precedence_table(g1, o1, l1, t1)
    show_precedence(p1, terms1)

    print("\n--- Example 2: Extended Expression Grammar ---")
    g2, o2 = parse_grammar("E -> E + T | E - T | T\nT -> T * F | T / F | F\nF -> ( E ) | id | num")
    print("Grammar:")
    for nt in o2:
        print(f"  {nt} -> {' | '.join(' '.join(p) for p in g2[nt])}")
    l2 = compute_leading(g2, o2)
    t2 = compute_trailing(g2, o2)
    show_sets(o2, l2, t2)
    p2, terms2 = build_precedence_table(g2, o2, l2, t2)
    show_precedence(p2, terms2)

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
        print("Grammar:")
        for nt in o:
            print(f"  {nt} -> {' | '.join(' '.join(p) for p in g[nt])}")
        lead = compute_leading(g, o)
        trail = compute_trailing(g, o)
        show_sets(o, lead, trail)
        p, terms = build_precedence_table(g, o, lead, trail)
        show_precedence(p, terms)


if __name__ == "__main__":
    main()
