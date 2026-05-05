def parse_grammar(text):
    grammar, order = {}, []
    for line in text.strip().split('\n'):
        line = line.strip()
        if not line or '->' not in line:
            continue
        lhs = line.split('->')[0].strip()
        rhs = line.split('->')[1].strip()
        grammar[lhs] = [alt.strip().split() for alt in rhs.split('|')]
        if lhs not in order:
            order.append(lhs)
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


def compute_first(grammar, order):
    nts = get_nts(grammar)
    first = {nt: set() for nt in nts}
    changed = True
    while changed:
        changed = False
        for nt in order:
            for prod in grammar[nt]:
                if prod == ['e']:
                    if 'e' not in first[nt]:
                        first[nt].add('e')
                        changed = True
                    continue
                all_eps = True
                for sym in prod:
                    if sym in nts:
                        new = first[sym] - {'e'}
                        if not new.issubset(first[nt]):
                            first[nt].update(new)
                            changed = True
                        if 'e' not in first[sym]:
                            all_eps = False
                            break
                    else:
                        if sym not in first[nt]:
                            first[nt].add(sym)
                            changed = True
                        all_eps = False
                        break
                if all_eps and 'e' not in first[nt]:
                    first[nt].add('e')
                    changed = True
    return first


def first_of_string(string, first, nts):
    result = set()
    if not string or string == ['e']:
        return {'e'}
    all_eps = True
    for sym in string:
        if sym in nts:
            result.update(first[sym] - {'e'})
            if 'e' not in first[sym]:
                all_eps = False
                break
        elif sym == 'e':
            continue
        else:
            result.add(sym)
            all_eps = False
            break
    if all_eps:
        result.add('e')
    return result


def compute_follow(grammar, order, first):
    nts = get_nts(grammar)
    follow = {nt: set() for nt in nts}
    follow[order[0]].add('$')
    changed = True
    while changed:
        changed = False
        for nt in order:
            for prod in grammar[nt]:
                for i, sym in enumerate(prod):
                    if sym not in nts:
                        continue
                    beta = prod[i + 1:]
                    if beta:
                        fb = first_of_string(beta, first, nts)
                        new = fb - {'e'}
                        if not new.issubset(follow[sym]):
                            follow[sym].update(new)
                            changed = True
                        if 'e' in fb and not follow[nt].issubset(follow[sym]):
                            follow[sym].update(follow[nt])
                            changed = True
                    else:
                        if not follow[nt].issubset(follow[sym]):
                            follow[sym].update(follow[nt])
                            changed = True
    return follow


def build_table(grammar, order, first, follow):
    nts = get_nts(grammar)
    terminals = get_terminals(grammar)
    terminals.add('$')
    table = {nt: {t: [] for t in terminals} for nt in order}
    is_ll1 = True

    for nt in order:
        for prod in grammar[nt]:
            fa = first_of_string(prod, first, nts)
            for t in fa:
                if t != 'e' and t in terminals:
                    if table[nt][t]:
                        is_ll1 = False
                    table[nt][t].append(prod)
            if 'e' in fa:
                for t in follow[nt]:
                    if t in terminals:
                        if table[nt][t]:
                            is_ll1 = False
                        table[nt][t].append(prod)

    return table, terminals, is_ll1


def show_table(table, order, terminals):
    tsorted = sorted(terminals - {'$'}) + ['$']
    w = 18
    print(f"\nLL(1) Parsing Table:")
    header = f"{'NT':<12}" + "".join(f"{t:<{w}}" for t in tsorted)
    print(header)
    print("-" * len(header))
    for nt in order:
        row = f"{nt:<12}"
        for t in tsorted:
            entries = table[nt].get(t, [])
            cell = ', '.join(f"{nt}->{' '.join(p)}" for p in entries) if entries else ""
            row += f"{cell:<{w}}"
        print(row)


def parse_string(inp, table, grammar, order):
    nts = get_nts(grammar)
    start = order[0]
    tokens = inp.strip().split() + ['$']
    stack = ['$', start]
    ptr, step = 0, 0

    print(f"\nParsing: {inp}")
    print(f"{'Step':<6}{'Stack':<30}{'Input':<25}{'Action':<25}")
    print("-" * 80)

    while stack:
        step += 1
        sstr = ' '.join(reversed(stack))
        rem = ' '.join(tokens[ptr:])
        top = stack[-1]
        cur = tokens[ptr]

        if top == '$' and cur == '$':
            print(f"{step:<6}{sstr:<30}{rem:<25}{'ACCEPTED':<25}")
            return True
        if top == cur:
            print(f"{step:<6}{sstr:<30}{rem:<25}{f'Match {top}':<25}")
            stack.pop()
            ptr += 1
        elif top in nts:
            if cur in table.get(top, {}) and table[top][cur]:
                prod = table[top][cur][0]
                action = f"{top} -> {' '.join(prod)}"
                print(f"{step:<6}{sstr:<30}{rem:<25}{action:<25}")
                stack.pop()
                if prod != ['e']:
                    for s in reversed(prod):
                        stack.append(s)
            else:
                print(f"{step:<6}{sstr:<30}{rem:<25}{'ERROR':<25}")
                return False
        else:
            print(f"{step:<6}{sstr:<30}{rem:<25}{'ERROR: mismatch':<25}")
            return False
    return False


def main():
    print("Predictive Parsing Table (LL(1) Parser)")
    print("-" * 40)

    print("\n--- Example 1: Expression Grammar ---")
    g1, o1 = parse_grammar("E -> T E'\nE' -> + T E' | e\nT -> F T'\nT' -> * F T' | e\nF -> ( E ) | id")
    print("Grammar:")
    for nt in o1:
        print(f"  {nt} -> {' | '.join(' '.join(p) for p in g1[nt])}")

    f1 = compute_first(g1, o1)
    fl1 = compute_follow(g1, o1, f1)

    print(f"\n{'NT':<12}{'FIRST':<25}{'FOLLOW':<25}")
    print("-" * 55)
    for nt in o1:
        print(f"  {nt:<10}{{ {', '.join(sorted(f1[nt]))} }}{'':<5}{{ {', '.join(sorted(fl1[nt]))} }}")

    t1, terms1, ll1 = build_table(g1, o1, f1, fl1)
    print(f"\nGrammar is {'LL(1)' if ll1 else 'NOT LL(1)'}")
    show_table(t1, o1, terms1)

    for s in ["id + id * id", "( id + id ) * id", "id * id + id"]:
        parse_string(s, t1, g1, o1)

    print("\n--- Example 2: Simple Grammar ---")
    g2, o2 = parse_grammar("S -> a B | c\nB -> b S | d")
    print("Grammar:")
    for nt in o2:
        print(f"  {nt} -> {' | '.join(' '.join(p) for p in g2[nt])}")
    f2 = compute_first(g2, o2)
    fl2 = compute_follow(g2, o2, f2)
    t2, terms2, ll2 = build_table(g2, o2, f2, fl2)
    show_table(t2, o2, terms2)
    parse_string("a b c", t2, g2, o2)

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
                l = l.replace('eps', 'e')
                lines.append(l)
        except EOFError:
            return
        if not lines:
            continue
        g, o = parse_grammar('\n'.join(lines))
        f = compute_first(g, o)
        fl = compute_follow(g, o, f)
        t, terms, is_ll = build_table(g, o, f, fl)
        print(f"Grammar is {'LL(1)' if is_ll else 'NOT LL(1)'}")
        show_table(t, o, terms)
        if is_ll:
            while True:
                try:
                    s = input("String to parse (or 'back'): ").strip()
                except EOFError:
                    return
                if s.lower() == 'back':
                    break
                if s:
                    parse_string(s, t, g, o)


if __name__ == "__main__":
    main()
