# Simple Typeahead Suggestions (using Trie)
The simplest, non-scalable feature (implemented using Trie data structure), that suggests next words of queries based on user input.

## This design is extremely flawed
Yes. From System-Design perspective, this can be very intersting design work. However, this current project only focuses on using basics of Trie functionality to implement the simplest possible typeahead sugestion feature. Hence this design project has following flaws, compared to a full-fledged typeahead / autocomplete feature used in search engines like Google, Bing etc.
 - Non-scalability
 - No suggestions based on geographical location of user
 - No suggestions based on current trends
 - No spell-check
 - Support for only English language


## Implementation
```java
import java.io.*;
import java.util.*;


public class AutoComplete {

	public static void main(String[] args) throws IOException {
		Trie trie = new Trie();
		System.out.println("Welcome to Simple Typeahead Suggestions System !!");
		System.out.println();
		System.out.println("Use below commands to interact with the program");
		System.out.println("Enter 0 to insert word");
		System.out.println("Enter 1 to get suggestions for text");
		System.out.println("Enter 2 to end the program");
		Scanner sc = new Scanner(System.in);
		while (true) {
			System.out.print("Enter command: ");
			String command = sc.next().trim();
			if (command.equals("0")) {
				System.out.println("Enter word to insert into dictionary");
				String word = sc.next().trim();
				trie.insertWord(word);
				System.out.println(word + " inserted");
				System.out.println();
			} else if (command.equals("1")) {
				System.out.println("Enter prefix to get suggestions from dictionary");
				String word = sc.next().trim();
				trie.autoComplete(word);
				System.out.println();
			} else if (command.equals("2")) {
				System.out.println("Program ended");
				break;
			} else {
				System.out.println("Invalid command");
				continue;
			}
		}
	}
}

class Node {

	HashMap<Character, Node> child;
	boolean endOfWord;
	int frequency;

	Node () {
		this.child = new HashMap<Character, Node>();
		this.endOfWord = false;
		this.frequency = 0;
	}

}


class Trie {

	Node root;

	final int k = 5;

	Trie () {
		this.root = new Node();
	}

	void insertWord (String word) {
		Node curr = root;
		for (int i = 0; i < word.length(); i++) {
			char curr_character = word.charAt(i);
			if (curr.child.containsKey(curr_character) == false) {
				curr.child.put(curr_character, new Node());
			}
			curr = curr.child.get(curr_character);
		}
		curr.endOfWord = true;
		curr.frequency += 1;
	}

	boolean searchWord (String word) {
		Node curr = root;
		for (int i = 0; i < word.length(); i++) {
			char curr_character = word.charAt(i);
			if (curr.child.containsKey(curr_character) == false) {
				return false;
			}
			curr = curr.child.get(curr_character);
		}
		return curr.endOfWord;
	}

	boolean searchPrefix (String prefix) {
		Node curr = root;
		for (int i = 0; i < prefix.length(); i++) {
			char curr_character = prefix.charAt(i);
			if (curr.child.containsKey(curr_character) == false) {
				return false;
			}
			curr = curr.child.get(curr_character);
		}
		if (curr.endOfWord) {
			curr.frequency += 1;
		}
		return true;
	}

	void autoComplete (String prefix) {
		boolean isPrefixPresent = searchPrefix(prefix);
		if (isPrefixPresent == false) {
			// insertWord(prefix);
			System.out.println(prefix);
			System.out.println("No extra suggestions for the word being searched first time !");
			return;
		} else {
			PriorityQueue<FrequencyPair> pq = new PriorityQueue<FrequencyPair>(k);
			getSuggestions(pq, prefix);
			if (pq.isEmpty() == false) {
				System.out.println("Top " + pq.size() + " typeahead suggestions are:");
				ArrayList<FrequencyPair> list = new ArrayList<>();
				while (pq.isEmpty() == false) {
					list.add(pq.poll());
				}
				Collections.sort(list, new customSort());
				for (FrequencyPair pair : list) {
					System.out.println(pair.word);
				}
			} else {
				System.out.println(prefix);
			}
			return;
		}
	}

	void getSuggestions (PriorityQueue<FrequencyPair> pq, String prefix) {
		Node curr = root;
		for (int i = 0; i < prefix.length(); i++) {
			char curr_character = prefix.charAt(i);
			curr = curr.child.get(curr_character);
		}
		for (char ch : curr.child.keySet()) {
			getSuggestionsUtil(curr.child.get(ch), prefix + ch, pq);
		}
	}

	void getSuggestionsUtil(Node node, String word, PriorityQueue<FrequencyPair> pq) {
		if (node.endOfWord) {
			if (pq.size() < k) {
				pq.add(new FrequencyPair(word, node.frequency));
			} else {
				if (pq.peek().frequency < node.frequency) {
					pq.poll();
					pq.add(new FrequencyPair(word, node.frequency));
				} else if (pq.peek().frequency == node.frequency) {
					if (pq.peek().word.compareTo(word) < 0) {
						pq.poll();
						pq.add(new FrequencyPair(word, node.frequency));
					}
				}
			}
		}
		for (char ch : node.child.keySet()) {
			getSuggestionsUtil(node.child.get(ch), word + ch, pq);
		}
	}

	class customSort implements Comparator <FrequencyPair> {
		public int compare (FrequencyPair a, FrequencyPair b) {
			if (a.frequency != b.frequency) {
				return (b.frequency - a.frequency);
			} else {
				return (a.word.compareTo(b.word));
			}
		}
	}
}

class FrequencyPair implements Comparable <FrequencyPair> {

	String word;
	int frequency;

	FrequencyPair (String word, int frequency) {
		this.word = word;
		this.frequency = frequency;
	}

	@Override
	public int compareTo (FrequencyPair frepair) {
		if (this.frequency != frepair.frequency) {
			return (this.frequency - frepair.frequency);
		} else {
			return (this.word.compareTo(frepair.word));
		}
	}
}
```
