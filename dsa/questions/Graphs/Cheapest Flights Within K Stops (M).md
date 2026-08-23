Previous: [[Core Graph Operations (LOL)]] | Next: [[Course Schedule (M)]]

There are n airports, labeled from 0 to n - 1, which are connected by some flights. You are given an array flights where flights[i] = [from_i, to_i, price_i] represents a one-way flight from airport from_i to airport to_i with cost price_i. You may assume there are no duplicate flights and no flights from an airport to itself.

You are also given three integers src, dst, and k where:

    src is the starting airport
    dst is the destination airport
    src != dst
    k is the maximum number of stops you can make (not including src and dst)

Return the cheapest price from src to dst with at most k stops, or return -1 if it is impossible.