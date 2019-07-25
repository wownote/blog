"=========== Meta ============
"StrID : 9
"Title : Example for markdown-pretty
"Slug  : example-for-markdown-pretty
"Cats  : 其它
"Tags  : 
"Date  : 20150130T02:44:28
"=============================
"EditType   : post
"EditFormat : Markdown
"========== Content ==========

# Example for markdown-pretty

Here is a small example for highlighting c code:

```c
#include <stdio.h>

/**
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */
int main(int argh, char* argv[]) {
    printf("Hello\n");
    return 0;
}
```

Here is a small example for highlighting java code:

```java
package com.google.mr4c;

/**
 * Copyright 2014 Google Inc. All rights reserved.
 * Unless required by applicable law or agreed to in writing.
 */
public class AlgoRunner {
    public static void main(String argv[]) throws Exception {
        AlgoRunnerConfig config = new AlgoRunnerConfig(true);
        config.setCommandLineArguments(argue);
        config.configure();
        AlgoRunner runner = new AlgoRunner(config);
        runner.executeStandalone();
    }
}
```

Here is a small example for highlighting python code:

```python
#!/usr/bin/env python
import os
import sys

if __name__ == "__main__":
    os.environ.setdefault("DJANGO_SETTINGS_MODULE", "example.settings")
    from django.core.management import execute_from_command_line
    execute_from_command_line(sys.argv)
}
```

Here is a small example for highlighting shell code:

```sh
nam install hero -g
```

done.

 
