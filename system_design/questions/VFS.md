```C++
#include <algorithm>
#include <iostream>
#include <memory>
#include <sstream>
#include <string>
#include <unordered_map>
#include <vector>

using namespace std;

class node {
      public:
        virtual ~node() = default;
        virtual int get_size() const = 0;
        virtual string get_name() const = 0;
        virtual bool is_dir() const = 0;
};

class file : public node {
      private:
        string name;
        string content;

      public:
        file(string s = "") : name(std::move(s)) {}
        bool is_dir() const override { return false; }
        string get_name() const override { return name; }
        int get_size() const override { return content.size(); }
        void add_content(const string &c) { content = c; };
        const string &get_content() const { return content; };
};

class directory : public node {
      private:
        string name;
        unordered_map<string, unique_ptr<node>> contents;

      public:
        directory(string n = "") : name(std::move(n)) {}
        bool is_dir() const override { return true; }
        string get_name() const override { return name; }
        int get_size() const override {
                int size = 0;
                for (auto &n : contents) {
                        size += n.second->get_size();
                }

                return size;
        }

        const unordered_map<string, unique_ptr<node>> &list_contents() const {
                return contents;
        };

        bool add_node(unique_ptr<node> nd) {
                if (contents.find(nd->get_name()) != contents.end()) {
                        return false;
                }
                contents[nd->get_name()] = std::move(nd);
                return true;
        }
};

class VFS {
      private:
        unique_ptr<directory> root;

        static vector<string> parse_path(const string &path) {
                stringstream ss{path};
                string token;
                vector<string> result;

                while (getline(ss, token, '/')) {
                        if (token != "") {
                                result.push_back(token);
                        }
                }

                return result;
        }

        static node *get_node(const vector<string> &path, directory &curr) {
                node *current = &curr;

                for (const auto &p : path) {
                        if (!current->is_dir()) {
                                return nullptr;
                        }

                        directory *curr_dir = static_cast<directory *>(current);
                        const auto &contents = curr_dir->list_contents();

                        auto it = contents.find(p);
                        if (it == contents.end()) {
                                return nullptr;
                        }

                        current = it->second.get();
                }

                return current;
        }

      public:
        VFS(const VFS &) = delete;
        VFS &operator=(const VFS &) = delete;
        VFS() : root(make_unique<directory>("/")) {}

        vector<string> ls(const string &path) {
                vector<string> parsed_path = parse_path(path);
                vector<string> result;
                node *f_node = get_node(parsed_path, *root);
                if (!f_node) {
                        cout << "ls: Path not found" << endl;
                        return {};
                }

                if (f_node->is_dir()) {
                        const auto &c =
                            static_cast<directory *>(f_node)->list_contents();

                        std::transform(c.begin(), c.end(),
                                       std::back_inserter(result),
                                       [](const auto &pair) {
                                               return pair.second->get_name();
                                       });

                } else {
                        result.push_back(f_node->get_name());
                }

                sort(result.begin(), result.end());

                for (const auto &r : result) {
                        cout << r << " ";
                }
                cout << endl;

                return result;
        }

        void mkdir(const string &path) {
                vector<string> parsed_path = parse_path(path);
                if (parsed_path.empty()) {
                        cout << "mkdir: No Input" << endl;
                        return;
                }

                string dir_name = parsed_path.back();
                parsed_path.pop_back();

                node *f_node = get_node(parsed_path, *root);
                if (!f_node) {
                        cout << "mkdir: Parent directory does not exist"
                             << endl;
                        return;
                }

                if (f_node->is_dir()) {
                        directory *f_dir = static_cast<directory *>(f_node);
                        if (!f_dir->add_node(
                                make_unique<directory>(dir_name))) {
                                cout
                                    << "mkdir: File or directory already exists"
                                    << endl;
                        }
                } else {
                        cout << "mkdir: Final not directory" << endl;
                }
        }

        void touch(const string &path) {
                vector<string> parsed_path = parse_path(path);
                if (parsed_path.empty()) {
                        cout << "touch: No Input" << endl;
                        return;
                }

                string file_name = parsed_path.back();
                parsed_path.pop_back();

                node *f_node = get_node(parsed_path, *root);
                if (!f_node) {
                        cout << "touch: Parent directory does not exist"
                             << endl;
                        return;
                }

                if (f_node->is_dir()) {
                        directory *f_dir = static_cast<directory *>(f_node);
                        if (!f_dir->add_node(make_unique<file>(file_name))) {
                                cout
                                    << "touch: File or directory already exists"
                                    << endl;
                        }
                } else {
                        cout << "touch: Final not directory" << endl;
                }
        }

        void write_file(const string &path, const string &content) {
                vector<string> parsed_path = parse_path(path);
                if (parsed_path.empty()) {
                        cout << "write_file: No Input Path" << endl;
                        return;
                }

                node *f_node = get_node(parsed_path, *root);
                if (!f_node) {
                        cout << "write_file: Target File does not exist"
                             << endl;
                        return;
                }

                if (f_node->is_dir()) {
                        cout << "write_file: trying to write to a "
                                "directory, invalid"
                             << endl;
                } else {
                        file *f_file = static_cast<file *>(f_node);
                        f_file->add_content(content);
                }
        }

        string read_file(const string &path) {
                vector<string> parsed_path = parse_path(path);
                if (parsed_path.empty()) {
                        cout << "read_file: No Input Path" << endl;
                        return "";
                }

                node *f_node = get_node(parsed_path, *root);
                if (!f_node) {
                        cout << "read_file: Target File does not exist" << endl;
                        return "";
                }

                if (f_node->is_dir()) {
                        cout << "read_file: trying to read a "
                                "directory, invalid"
                             << endl;
                        return "";
                }

                return static_cast<file *>(f_node)->get_content();
        }
};

int main() {

        VFS vfs;
        vfs.ls("");
        vfs.mkdir("/bin");
        vfs.mkdir("/etc");
        vfs.ls("/");
        vfs.mkdir("/bin/network");
        vfs.ls("/bin");
        vfs.touch("/bin/network/ip_list");
        vfs.ls("/bin/network");

        vfs.write_file("/bin/network/ip_list", "192.168.1.1");
        cout << "Content: " << vfs.read_file("/bin/network/ip_list") << endl;

        return 0;
}

```