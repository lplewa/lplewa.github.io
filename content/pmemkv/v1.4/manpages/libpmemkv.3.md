---
draft: false
layout: "library"
slider_enable: true
description: ""
disclaimer: "The contents of this web site and the associated <a href=\"https://github.com/pmem\">GitHub repositories</a> are BSD-licensed open source."
aliases: ["libpmemkv.3.html"]
title: "libpmemkv | PMDK"
---

[comment]: <> (SPDX-License-Identifier: BSD-3-Clause)
[comment]: <> (Copyright 2019-2021, Intel Corporation)

[comment]: <> (libpmemkv.3 -- man page for libpmemkv)

[NAME](#name)<br />
[SYNOPSIS](#synopsis)<br />
[DESCRIPTION](#description)<br />
[ERRORS](#errors)<br />
[EXAMPLE](#example)<br />
[SEE ALSO](#see-also)<br />


# NAME #

**pmemkv** - Key/Value Datastore for Persistent Memory

# SYNOPSIS #

```c
#include <libpmemkv.h>

typedef int pmemkv_get_kv_callback(const char *key, size_t keybytes, const char *value,
			size_t valuebytes, void *arg);
typedef void pmemkv_get_v_callback(const char *value, size_t valuebytes, void *arg);

int pmemkv_open(const char *engine, pmemkv_config *config, pmemkv_db **db);
void pmemkv_close(pmemkv_db *kv);

int pmemkv_count_all(pmemkv_db *db, size_t *cnt);
int pmemkv_count_above(pmemkv_db *db, const char *k, size_t kb, size_t *cnt);
int pmemkv_count_below(pmemkv_db *db, const char *k, size_t kb, size_t *cnt);
int pmemkv_count_between(pmemkv_db *db, const char *k1, size_t kb1, const char *k2,
			size_t kb2, size_t *cnt);

int pmemkv_get_all(pmemkv_db *db, pmemkv_get_kv_callback *c, void *arg);
int pmemkv_get_above(pmemkv_db *db, const char *k, size_t kb, pmemkv_get_kv_callback *c,
			void *arg);
int pmemkv_get_below(pmemkv_db *db, const char *k, size_t kb, pmemkv_get_kv_callback *c,
			void *arg);
int pmemkv_get_between(pmemkv_db *db, const char *k1, size_t kb1, const char *k2,
			size_t kb2, pmemkv_get_kv_callback *c, void *arg);

int pmemkv_exists(pmemkv_db *db, const char *k, size_t kb);

int pmemkv_get(pmemkv_db *db, const char *k, size_t kb, pmemkv_get_v_callback *c,
			void *arg);
int pmemkv_get_copy(pmemkv_db *db, const char *k, size_t kb, char *buffer,
			size_t buffer_size, size_t *value_size);
int pmemkv_put(pmemkv_db *db, const char *k, size_t kb, const char *v, size_t vb);

int pmemkv_remove(pmemkv_db *db, const char *k, size_t kb);

int pmemkv_defrag(pmemkv_db *db, double start_percent, double amount_percent);

const char *pmemkv_errormsg(void);
```

For pmemkv configuration API description see **libpmemkv_config**(3).
For pmemkv iterator API description see **libpmemkv_iterator**(3).
For general pmemkv information, engine descriptions and bindings details see **libpmemkv**(7).

# DESCRIPTION #

Keys and values stored in a pmemkv database can be arbitrary binary data and can contain multiple null characters.
Every function which accepts key expects `const char *k` pointer to data and its size as `size_t`.

Some of the functions (mainly range-query API) are not guaranteed to be implemented by all engines.
If an engine does not support a certain function, it will return PMEMKV\_STATUS\_NOT\_SUPPORTED.

Note: There are no explicit upper_bound/lower_bound functions. If you want to obtain an element(s)
above or below the selected key, you can use pmemkv_get_above() or pmemkv_get_below().
See descriptions of these functions for details.

`int pmemkv_open(const char *engine, pmemkv_config *config, pmemkv_db **db);`

:	Opens the pmemkv database and stores a pointer to a *pmemkv_db* instance in `*db`.
	The `engine` parameter specifies the engine name (see **libpmemkv**(7) for the list of available engines).
	The `config` parameter specifies configuration (see **libpmemkv_config**(3) for details). Pmemkv takes
	ownership of the config parameter - this means that pmemkv_config_delete() must NOT be called
	after open (successful or failed).

`void pmemkv_close(pmemkv_db *kv);`

:	Closes pmemkv database.

`int pmemkv_count_all(pmemkv_db *db, size_t *cnt);`

:	Stores in `*cnt` the number of records in `db`.

`int pmemkv_count_above(pmemkv_db *db, const char *k, size_t kb, size_t *cnt);`

:	Stores in `*cnt` the number of records in `db` whose keys are greater than
	the key `k` of length `kb`. Order of the elements is specified by a comparator (see **libpmemkv**(7)).

`int pmemkv_count_below(pmemkv_db *db, const char *k, size_t kb, size_t *cnt);`

:	Stores in `*cnt` the number of records in `db` whose keys are less than
	the key `k` of length `kb`. Order of the elements is specified by a comparator (see **libpmemkv**(7)).

`int pmemkv_count_between(pmemkv_db *db, const char *k1, size_t kb1, const char *k2, size_t kb2, size_t *cnt);`

:	Stores in `*cnt` the number of records in `db` whose keys are greater than key `k1` (of length `kb1`)
	and less than key `k2` (of length `kb2`). Order of the elements is specified by a comparator (see **libpmemkv**(7)).

`int pmemkv_get_all(pmemkv_db *db, pmemkv_get_kv_callback *c, void *arg);`

:	Executes function `c` for every record stored in `db`. Arguments
	passed to the function are: pointer to a key, size of the key, pointer to a value, size of
	the value and `arg` specified by the user.
	Function `c` can stop iteration by returning non-zero value. In that case *pmemkv_get_all()* returns
	PMEMKV\_STATUS\_STOPPED\_BY\_CB. Returning 0 continues iteration.
	Order of the elements is specified by a comparator (see **libpmemkv**(7)).

`int pmemkv_get_above(pmemkv_db *db, const char *k, size_t kb, pmemkv_get_kv_callback *c, void *arg);`

:	Executes function `c` for every record stored in `db` whose keys are greater than
	key `k` (of length `kb`). Arguments passed to `c` are: pointer to a key, size of the key, pointer to a value, size of
	the value and `arg` specified by the user.
	Function `c` can stop iteration by returning non-zero value. In that case *pmemkv_get_above()* returns
	PMEMKV\_STATUS\_STOPPED\_BY\_CB. Returning 0 continues iteration.
	Order of the elements is specified by a comparator (see **libpmemkv**(7)).

`int pmemkv_get_below(pmemkv_db *db, const char *k, size_t kb, pmemkv_get_kv_callback *c, void *arg);`

:	Executes function `c` for every record stored in `db` whose keys are less than
	key `k` (of length `kb`). Arguments passed to `c` are: pointer to a key, size of the key, pointer to a value, size of
	the value and `arg` specified by the user.
	Function `c` can stop iteration by returning non-zero value. In that case *pmemkv_get_below()* returns
	PMEMKV\_STATUS\_STOPPED\_BY\_CB. Returning 0 continues iteration.
	Order of the elements is specified by a comparator (see **libpmemkv**(7)).

`int pmemkv_get_between(pmemkv_db *db, const char *k1, size_t kb1, const char *k2, size_t kb2, pmemkv_get_kv_callback *c, void *arg);`

:	Executes function `c` for every record stored in `db` whose keys are greater than
	key `k1` (of length `kb1`) and less than key `k2` (of length `kb2`).
	Arguments passed to `c` are: pointer to a key, size of the key, pointer to a value, size of
	the value and `arg` specified by the user.
	Function `c` can stop iteration by returning non-zero value. In that case *pmemkv_get_between()* returns
	PMEMKV\_STATUS\_STOPPED\_BY\_CB. Returning 0 continues iteration.
	Order of the elements is specified by a comparator (see **libpmemkv**(7)).

`int pmemkv_exists(pmemkv_db *db, const char *k, size_t kb);`

:	Checks existence of record with key `k` of length `kb`.
	If record is present PMEMKV\_STATUS\_OK is returned, otherwise PMEMKV\_STATUS\_NOT\_FOUND is returned.
	Other possible return values are described in the *ERRORS* section.

`int pmemkv_get(pmemkv_db *db, const char *k, size_t kb, pmemkv_get_v_callback *c, void *arg);`

:	Executes function `c` on record with key `k` (of length `kb`).
	If record is present and no error occurred the function returns PMEMKV\_STATUS\_OK.
	If record does not exist PMEMKV\_STATUS\_NOT\_FOUND is returned.
	Other possible return values are described in the *ERRORS* section.
	Function `c` is called with the following parameters: pointer to a value, size of the value and `arg` specified by the user.
	`Value` points to the location where data is actually stored (no copy occurs).
	This function is guaranteed to be implemented by all engines.

`int pmemkv_get_copy(pmemkv_db *db, const char *k, size_t kb, char *buffer, size_t buffer_size, size_t *value_size);`

:	Copies value of record with key `k` of length `kb` to user provided buffer.
	`buffer` points to the value buffer, `buffer_size` specifies its size and `*value_size`
	is filled in by this function. If the value doesn't fit in the provided buffer
	then this function returns PMEMKV\_STATUS\_UNKNOWN\_ERROR.
	Otherwise, in absence of any errors, PMEMKV\_STATUS\_OK is returned.
	Other possible return values are described in the *ERRORS* section.
	This function is guaranteed to be implemented by all engines.

`int pmemkv_put(pmemkv_db *db, const char *k, size_t kb, const char *v, size_t vb);`

:	Inserts a key-value pair into pmemkv database. `kb` is the length of key `k` and `vb` is the length of value `v`.
	When this function returns, caller is free to reuse both buffers.
	This function is guaranteed to be implemented by all engines.

`int pmemkv_remove(pmemkv_db *db, const char *k, size_t kb);`

:	Removes record with key `k` of length `kb`.
	This function is guaranteed to be implemented by all engines.

`int pmemkv_defrag(pmemkv_db *db, double start_percent, double amount_percent);`

:	Defragments approximately 'amount_percent' percent of elements in the database
	starting from 'start_percent' percent of elements.

`const char *pmemkv_errormsg(void);`

:	Returns a human readable string describing the last error.

## ERRORS ##

Each function, except for *pmemkv_close()* and *pmemkv_errormsg()*, returns one of the following status codes:

+ **PMEMKV_STATUS_OK** -- no error
+ **PMEMKV_STATUS_UNKNOWN_ERROR** -- unknown error
+ **PMEMKV_STATUS_NOT_FOUND** -- record not found
+ **PMEMKV_STATUS_NOT_SUPPORTED** -- function is not implemented by current engine
+ **PMEMKV_STATUS_INVALID_ARGUMENT** -- argument to function has wrong value
+ **PMEMKV_STATUS_CONFIG_PARSING_ERROR** -- parsing data to config failed
+ **PMEMKV_STATUS_CONFIG_TYPE_ERROR** -- config item has different type than expected
+ **PMEMKV_STATUS_STOPPED_BY_CB** -- iteration was stopped by user's callback
+ **PMEMKV_STATUS_OUT_OF_MEMORY** -- operation failed because there is not enough memory (or space on the device)
+ **PMEMKV_STATUS_WRONG_ENGINE_NAME** -- engine name does not match any available engine
+ **PMEMKV_STATUS_TRANSACTION_SCOPE_ERROR** -- an error with the scope of the libpmemobj transaction
+ **PMEMKV_STATUS_DEFRAG_ERROR** -- the defragmentation process failed (possibly in the middle of a run)

Status returned from a function can change in a future version of a library to a more specific one.
For example, if a function returns PMEMKV_STATUS_UNKNOWN_ERROR, it is possible that in future
versions it will return PMEMKV_STATUS_INVALID_ARGUMENT. Recommended way to check for an error
is to compare status with PMEMKV_STATUS_OK.

# EXAMPLE #

The following example is taken from `examples/pmemkv_basic_c` directory.

Basic pmemkv usage in C:

```c
#include <assert.h>
#include <libpmemkv.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#define ASSERT(expr)                                                                     \
	do {                                                                             \
		if (!(expr))                                                             \
			puts(pmemkv_errormsg());                                         \
		assert(expr);                                                            \
	} while (0)

#define LOG(msg) puts(msg)
#define MAX_VAL_LEN 64

static const uint64_t SIZE = 1024UL * 1024UL * 1024UL;

int get_kv_callback(const char *k, size_t kb, const char *value, size_t value_bytes,
		    void *arg)
{
	printf("   visited: %s\n", k);

	return 0;
}

int main(int argc, char *argv[])
{
	if (argc < 2) {
		fprintf(stderr, "Usage: %s file\n", argv[0]);
		exit(1);
	}

	/* See libpmemkv_config(3) for more detailed example of config creation */
	LOG("Creating config");
	pmemkv_config *cfg = pmemkv_config_new();
	ASSERT(cfg != NULL);

	int s = pmemkv_config_put_path(cfg, argv[1]);
	ASSERT(s == PMEMKV_STATUS_OK);
	s = pmemkv_config_put_size(cfg, SIZE);
	ASSERT(s == PMEMKV_STATUS_OK);
	s = pmemkv_config_put_force_create(cfg, true);
	ASSERT(s == PMEMKV_STATUS_OK);

	LOG("Opening pmemkv database with 'cmap' engine");
	pmemkv_db *db = NULL;
	s = pmemkv_open("cmap", cfg, &db);
	ASSERT(s == PMEMKV_STATUS_OK);
	ASSERT(db != NULL);

	LOG("Putting new key");
	const char *key1 = "key1";
	const char *value1 = "value1";
	s = pmemkv_put(db, key1, strlen(key1), value1, strlen(value1));
	ASSERT(s == PMEMKV_STATUS_OK);

	size_t cnt;
	s = pmemkv_count_all(db, &cnt);
	ASSERT(s == PMEMKV_STATUS_OK);
	ASSERT(cnt == 1);

	LOG("Reading key back");
	char val[MAX_VAL_LEN];
	s = pmemkv_get_copy(db, key1, strlen(key1), val, MAX_VAL_LEN, NULL);
	ASSERT(s == PMEMKV_STATUS_OK);
	ASSERT(!strcmp(val, "value1"));

	LOG("Iterating existing keys");
	const char *key2 = "key2";
	const char *value2 = "value2";
	const char *key3 = "key3";
	const char *value3 = "value3";
	pmemkv_put(db, key2, strlen(key2), value2, strlen(value2));
	pmemkv_put(db, key3, strlen(key3), value3, strlen(value3));
	pmemkv_get_all(db, &get_kv_callback, NULL);

	LOG("Removing existing key");
	s = pmemkv_remove(db, key1, strlen(key1));
	ASSERT(s == PMEMKV_STATUS_OK);
	ASSERT(pmemkv_exists(db, key1, strlen(key1)) == PMEMKV_STATUS_NOT_FOUND);

	LOG("Defragmenting the database");
	s = pmemkv_defrag(db, 0, 100);
	ASSERT(s == PMEMKV_STATUS_OK);

	LOG("Closing database");
	pmemkv_close(db);

	return 0;
}

```

## Common usage mistake ##

Common mistake in pmemkv API usage (especially when using C++ API) is to dereference
pointer to the data stored in pmemkv outside of a callback function scope.

```cpp
std::string value;
const char* ptr;
size_t sz;
kv->get("key1", [&](string_view v) {
	/* Save pointer to the data to use it later outside of a callback scope */
	ptr = v.data();
	sz = v.size();
});
kv->remove("key");
/* ERROR!
 * Using this pointer outside of a callback function may cause access to some random data
 * or a segmentation fault. At that point, ptr should be considered as invalid.
 */
value.append(ptr, sz);
```

# SEE ALSO #

**libpmemkv**(7), **libpmemkv_config**(3), **libpmemkv_iterator**(3) and **<https://pmem.io>**
