# logging levels
Logging has several levels that differ in how much the info is critical. Which one is the default?

> DEBUG    : detailed analysis
> INFO     : confirmation that all is working well
> WARNING  : default! smt unexpected, but still working
> ERROR    : smt does not work
> CRITICAL : the whole program unable to continue

# simple logging
Using logging directly: this will set the logger's name to 'root'. The first time you define its configuration will be the one that is used: for more flexibility you should use logger instances. 

```python
logging.basicConfig(
    filename='test.log',                           # avoid the console
    level=logging.DEBUG,
    format='%(asctime)s:%(levelname)s:%(message)s' # format of the msg
)
logging.debug('message')  # logs the string
```

# logger instances
This is the preferred option when using modules. You can set the logging level for each module separately.

```python
logger = logging.getLogger(__name__)  # name equal to module name or "__main__"
logger.setLevel(logging.INFO)  # can set levels for handlers separately, but the main logger sets the max level

formatter = logging.Formatter('%(asctime)s:%(levelname)s:%(message)s')

file_handler = logging.FileHandler('test.log')
file_handler.setFormatter(formatter)  # alt StreamHandler, email handler, etc
logger.addHandler(file_handler)

logger.info('an info message')
```

# include traceback
The method below will log the traceback.

```python
logging.exception('message')  # or using a logger instance
```