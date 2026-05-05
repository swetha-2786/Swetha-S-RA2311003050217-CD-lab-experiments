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


def get_non_terminals(grammar):
    return set(grammar.keys())


def get_terminals(grammar):
    nts = get_non_terminals(grammar)
    terms = set()
    for nt in grammar:
        for prod in grammar[nt]:
            for sym in prod:
                if sym not in nts and sym != 'e':
                    terms.add(sym)
    return terms


def compute_first(grammar, order):
    nts = get_non_terminals(grammar)
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
                        added = first[sym] - {'e'}
                        if not added.issubset(first[nt]):
                            first[nt].update(added)
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
    nts = get_non_terminals(grammar)
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
                        added = fb - {'e'}
                        if not added.issubset(follow[sym]):
                            follow[sym].update(added)
                            changed = True
                        if 'e' in fb and not follow[nt].issubset(follow[sym]):
                            follow[sym].update(follow[nt])
                            changed = True
                    else:
                        if not follow[nt].issubset(follow[sym]):
                            follow[sym].update(follow[nt])
                            changed = True
    return follow


def display_results(order, first, follow):
    print(f"\n{'Non-Terminal':<15} {'FIRST':<25} {'FOLLOW':<25}")
    print("-" * 60)
    for nt in order:
        f_str = "{ " + ", ".join(sorted(first[nt])) + " }"
        fl_str = "{ " + ", ".join(sorted(follow[nt])) + " }"
        print(f"  {nt:<13} {f_str:<25} {fl_str:<25}")


def main():
    print("FIRST and FOLLOW Set Computation")
    print("-" * 40)

    print("\n--- Example 1: Expression Grammar ---")
    g1, o1 = parse_grammar("E -> T E'\nE' -> + T E' | e\nT -> F T'\nT' -> * F T' | e\nF -> ( E ) | id")
    print("Grammar:")
    for nt in o1:
        print(f"  {nt} -> {' | '.join(' '.join(p) for p in g1[nt])}")
    f1 = compute_first(g1, o1)
    fl1 = compute_follow(g1, o1, f1)
    display_results(o1, f1, fl1)

    print("\n--- Example 2: Simple Grammar ---")
    g2, o2 = parse_grammar("S -> A B C\nA -> a | e\nB -> b | e\nC -> c | e")
    print("Grammar:")
    for nt in o2:
        print(f"  {nt} -> {' | '.join(' '.join(p) for p in g2[nt])}")
    f2 = compute_first(g2, o2)
    fl2 = compute_follow(g2, o2, f2)
    display_results(o2, f2, fl2)

    print("\n--- Example 3: Complex Dependencies ---")
    g3, o3 = parse_grammar("S -> a B D h\nB -> c C\nC -> b C | e\nD -> E F\nE -> g | e\nF -> f | e")
    print("Grammar:")
    for nt in o3:
        print(f"  {nt} -> {' | '.join(' '.join(p) for p in g3[nt])}")
    f3 = compute_first(g3, o3)
    fl3 = compute_follow(g3, o3, f3)
    display_results(o3, f3, fl3)

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
        print("Grammar:")
        for nt in o:
            print(f"  {nt} -> {' | '.join(' '.join(p) for p in g[nt])}")
        f = compute_first(g, o)
        fl = compute_follow(g, o, f)
        display_results(o, f, fl)


if __name__ == "__main__":
    main()
