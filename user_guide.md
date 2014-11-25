# VUnit User Guide
# Python Interface

The public interface of VUnit is exposed through the `VUnit` class
that can be imported directly from the vunit module.

```python
# file: run.py
import vunit

# Create VUnit instance by parsing command line arguments
vu = VUnit.from_argv()

# Create library 'lib' 
lib = vu.add_library("lib")

# Add all files ending in .vhd in current working directory 
lib.add_source_files("*.vhd")

# Run vunit function
vu.main()
```

A `VUnit` object can be created from command line arguments by using
the `from_argv` method effectively creating a custom command line tool
for running tests in the user project.  Source files and libraries are
added to the project by using methods on the VUnit object. The
configuration is followed by a call to the `main` method which will
execute the function specified by the command line arguments and exit
the script. The added source files are automatically scanned for test
cases.

```shell
> python run.py -h
usage: run.py [-h] [-l] [--compile] [--clean] [-o OUTPUT_PATH] [-x XUNIT_XML]
              [-v] [--no-color] [--log-level {info,error,warning,debug}]
              [tests [tests ...]]

VUnit command line tool.

positional arguments:
  tests                 Tests to run

optional arguments:
  -h, --help            show this help message and exit
  -l, --list            Only list all test cases
  --compile             Only compile project
  --clean               Remove output path first
  -o OUTPUT_PATH, --output-path OUTPUT_PATH
                        Output path for compilation and simulation artifacts
  -x XUNIT_XML, --xunit-xml XUNIT_XML
                        Xunit test report .xml file
  -v, --verbose         Print test output immediately and not only when
                        failure
  --no-color            Do not color output
  --log-level {info,error,warning,debug}
```

## VHDL Test Benches
In its simplest form a VUnit VHDL test bench looks like this:
```vhdl
-- file tb_example.vhd
library vunit_lib;
context vunit_lib.vunit_context;

entity tb_example is
  generic (runner_cfg : runner_cfg_t);
end entity;

architecture tb of tb_example is
begin
  main : process
  begin
    test_runner_setup(runner, runner_cfg);
    report "Hello world!";
    test_runner_cleanup(runner); -- Simulation ends here
  end process;
end architecture;
```

From `tb_example.vhd` a single test case named `lib.tb_example` is
created.  It is also possible to put multiple tests in a single test
benchs that are each run in individual simulations. Putting multiple
tests in the same test bench is a good way to share a common test
environment.

```vhdl
-- file tb_example_many.vhd
library vunit_lib;
context vunit_lib.vunit_context;

entity tb_example_many is
  generic (runner_cfg : runner_cfg_t);
end entity;

architecture tb of tb_example_many is
begin
  main : process
  begin
    test_runner_setup(runner, runner_cfg);
    
    while test_suite loop
      
      if run("test_pass") then
        report "This will pass";
        
      elsif run("test_fail") then
        assert false report "It fails";
        
      end if;
    end loop;
    
    test_runner_cleanup(runner);
  end process;
end architecture;
```

From `tb_example_many.vhd` two test cases named
`lib.tb_example_many.test_pass` and `tb_example_many.test_fail` are
created.

We can run these test using `run.py` command line interface:
```shell
> python run.py -v lib.tb_example*
Running test: lib.tb_example
Running test: lib.tb_example_many.test_pass
Running test: lib.tb_example_many.test_fail
Running 3 tests
running lib.tb_example
Hello World!
pass (P=1 F=0 T=3) lib.tb_example

running lib.tb_example.test_pass
This will pass
pass (P=2 F=0 T=3) lib.tb_example_many.test_pass

running lib.tb_example.test_fail
Error: It fails
fail (P=2 F=1 T=3) lib.tb_example_many.test_fail

pass lib.tb_example after 0.1 seconds
pass lib.tb_example_many.test_pass after 0.1 seconds
fail lib.tb_example_many.test_fail after 0.1 seconds

Total time 0.3 seconds
2 of 3 passed
Some failed!
```

The above example code can be found in `examples/user_guide/`.