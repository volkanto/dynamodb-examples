# Some DynamoDB examples

* v1
  * start the localstack docker image with DynamoDB before the test
  * first try: create table, use global secondary index, annotate the order entity
  * crud and read by GSI
  * no error on creating duplicate orders
* v2
  * in-memory DynamoDB (run mvn test-compile once to create the `native-libs` folder containing the so and dll files 
  for sqlite used by DynamoDB)
  * error on creating duplicate orders
* v3
  * primary key of GSI does not need to be unique, so we have to assert uniqueness of (pOID, global-key) differently
* v4
  * use 2 order tables to assert uniqueness of (pOID, gEID) and (pOID, gK)
  * use insert with TX to throw error on duplicates
  * started with vendor example: use optimistic locking with versioning when applying changes via listener
    * handle "duplicate" orders for same rps vendor id at the same time!
    * don't use big TX for "read all vendors for rpsId, delete them and add again according to message" but handle each
    entry on its own (using optimistic locking and the existing and provided timestamps)
    * not finished yet (don't even know, if the approach works)
* v5
  * played around with different approaches how to update the vendors in the listener via queue message
  * versioning, conditional expressions
  * but each platform vendor updated singularly
  * does not look promising
* v6
  * TX approach for updating the vendors
  * all-or-nothing with retries
  * TX can fail due to concurrent modification or platform vendor already exists (due to duplicate message or for other 
  rps vendor id) or queue message timestamp outdated
  * looks more promising, but still thinking to do