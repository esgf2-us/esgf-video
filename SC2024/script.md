# Intro

# Intake-esgf

Read in 3:12

- The following provides an introduction to intake-esgf: a python package providing a programmatic interface to the ESGF holdings.
- ESGF holdings are vast and so our catalogs initialize empty. They are populated using the catalog search function and providing facets or keywords with search criteria. In this example we are searching for monthly mean output from the Canadian model that was run from 1850 to 2015. We would like the gross primary productivty, surface air temperature, and precipitation variables. This function queries a configurable set of indices and returns merged results in the form of a pandas dataframe.
- This dataframe is directly accessible to the user as a property of the catalog object. The user can then manipulate the dataframe by removing rows in order to hone the search to some desired selection.
- Alternatively, the user can print the catalog to view the columns and unique values of the underlying dataframe, utilizing this information to refine the search.
- In this case, we realize that we neglected to specify the ensemble member. We repeat our search with this additional constraint which leads to our intended result. 
- Once the catalog contains the desired datasets, they may be loaded into memory with the to_dataset_dict() catalog function. Internally we are communicating again with the index nodes to consolidate access methods and locations for all the files that are part of each dataset.
- We use the following logic to determine how we will transfer or load your datasets. First, if a file already exists locally, we will simply load it. This may be because the file is already in the cache from a previous use of intake-esgf. It may also be because the user is working on a resource with direct access to the file. Second, if the user has indicated a preference to stream data and such access methods are available, we will harvest and provide these paths. Third, if the user has a destination Globus endpoint to which they prefer to download the data, they may provide the UUID to initiate a Globus transfer. As a last resort, we will download the remaining files using a threaded https transfer.
- The to_dataset_dict() function returns a dictionary of datasets whose keys are formed from the different facet values of the datasets returned. In this case it uses the table_id and the variable_id. Note that intake-esgf also automatically includes cell measures which in this case is areacella.
- As a demonstration, we present a simple python code to plot the temporal mean of each variable using matplotlib. From left to right we plot the temperature, precipitation, and gross primary productivity.
- The programmatic nature of the interface is important because it allows the user to have a conversation with the index and learn what data may be appropriate to their science question. It also hides the complexity about how the data is stored or made available to the user and automates many painful tasks. The interface also makes user scripts portable across systems, particularly important as server-side and data-proximate computing options become available.

# Rooki 

Read in 2:30

- One way we can use intake-esgf to enable server-side computing is through the python package rooki.
- First we import what we will need for this demonstration. This includes the rooki package which requires that we have an environment variable set prior to import. The ROOK_URL variable points to the web processing service that we will be using.
- Because we are using the ORNL WPS deployment, we need to ensure intake-esgf only finds records of things located at ORNL. This we do in 2 ways. First, by configuring the indices that will be queried...
- ...but also by specifying the data_node when performing a catalog search. As a demonstration we look for historical temperature data from four models.
- In order for rooki to remotely load a dataset, it needs an id that we can construct from the dataset_ids that are returned in the `id` column of the catalog dataframe. Here we write a function that uses the first id from the list of ids per dataset, removes the data location, and then prepends part of the local path. We then apply this function to the dataframe column to obtain a list of rooki ids that we can pass to the WPS.
- Rooki provides the user with a set of operators that can be nested to perform a sequence of operations server-side. Here we write a function that uses an input rooki id to load the temperature data, subset in time from 1990 to 2000 and in space using a latitude and longitude bounding box, and then average annually in time. Once a workflow is defined, we ask rooki to orchestrate it which will return the reduced datasets if no errors were encountered.
- We use this function to operate on each of our rooki ids, loading the returned datasets into a dictionary whose keys are the model names. As our workflow is executed on the server, we will observe messages indicating that the reduced result files have been transferred to our local system.
- As a demonstration of functionality, we loop over each dataset and plot the mean annual temperature of 1995 from each model using matplotlib.
- Rooki is a great solution if your task fits into the operators that they implement. This minimizes data transfer loads by reducing the data before sending it to the user. We used intake-esgf to help form the ids that rooki needs to remotely load data.

# Globus Compute 

Read in 3:06

- A more flexible option is to use Globus Compute.
- What if we have a computation that fits outside the averaging/subsetting/regridding workflows? For this demonstration we will consider the el Nino southern oscillation index. This is an important climate index built by combining some statistical measures of model output around the equator.
- Globus Compute can handle this custom workflow. It provides a mechanism to supply credentials, configure and balance our workload and execution environment, and retrieve the results of the computation.
- So, given a target workload, how do we start using Globus Compute?
- First we write a function that uses intake-esgf to load data and then perform the desired operations. This function will need to be registered with Globus.
- Once registered, you can share your function with a user group of your choosing via globus.org.
- Now we will demonstrate this with the ENSO example.
- We start by importing packages we will use for computation and visualization, using intake-esgf for data access.
- Then we write, register, and test our function. Make sure that all libraries are imported in your function and also installed. The output of the function also needs to be serializable. In our example we will compute the ENSO index for 2 models, CESM2 and MIROC6.
- Our function takes a source_id as input, uses intake-esgf to locate and load data, and then compute the ENSO index in an xarray dataset.
- We first test it locally by passing in a source_id, CESM2, and then inspecting the returned dataset.
- Once the function is working, we need to initialize our compute endpoint. This requires that globus-compute-endpoint is installed, after which we run configure to provide a endpoint name and start to return a ID.
- We will use this ID and so we store it now.
- Next we setup an executor which uses this ID and requires that ports are configured. We are now ready to compute.
- We launch our ENSO function using Globus compute on our endpoint. We do this by submitting 2 tasks to the endpoint, in each case supplying the source_id.
- In order to pull down results, we have to call each task's result function, storing each in a local variable.
- Once we have these datasets, now we can visualize locally. We have written a plotting function which we can call on both result datasets to study the ENSO metric through the years.
- Now we can share this function with others with Globus Groups. We have created a group for ESGF work at the ALCF. We only need to copy the groups ID.
- Then we use the Globus client to register this function. Providing the group ID allows all group members to discover and launch the function. This will return a function ID to which we will refer.
- We can run this function as before, providing a compute endpoint as well as the function id we just created. Running this function returns the same result as before.
- Globus compute is a great solution if a user needs custom computation near the data. As with rooki, it also minimizes data transfer by only returning the results. In this case, intake-esgf was used to locate the data that already exists on the remote system, avoiding unnecessary downloads.
