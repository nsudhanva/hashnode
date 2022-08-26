## Analyzing Data For A Food Tech Startup - Swiggy

Swiggy Data Science Assessment - LinkedIn, MTV Get a Job
========================================================

    import numpy as np
    import pandas as pd
    import matplotlib.pyplot as plt
    from scipy import stats
    from sklearn.cluster import KMeans
    from sklearn.model_selection import train_test_split
    from sklearn.preprocessing import StandardScaler
    from sklearn.decomposition import PCA
    

Import Dataset and Quick Inspection
-----------------------------------

    dataset_path = './data/SampleAssessment.csv'
    df = pd.read_csv(dataset_path)
    

    df.head()
    

    df.tail()
    

    df['first_time'] = pd.to_datetime(df['first_time'])
    df['recent_time'] = pd.to_datetime(df['recent_time'])
    

    df.head()
    

Number of missing values (orders in 7 days and last 4 weeks)
------------------------------------------------------------

    df.isna().sum()
    

    customer_id               0
    first_time                0
    recent_time               0
    no_of_orders              0
    orders_7_days          8077
    orders_last_4_weeks    5659
    amount                    0
    amt_last_7_days           0
    amt_in_last_4_weeks       0
    avg_dist_rest             0
    avg_del_time              0
    dtype: int64
    

Verifying whether the amount is zero for no orders
--------------------------------------------------

    (df['amt_last_7_days'] == 0).sum(axis=0)
    

    8078
    

    (df['amt_in_last_4_weeks'] == 0).sum(axis=0)
    

    5660
    

Description of the subset of dataset with negative restaurant distances
-----------------------------------------------------------------------

    df[df['avg_dist_rest'] < 0].describe().round(2)
    

Description of the original dataset with negative restaurant distances
----------------------------------------------------------------------

    df.describe().round(2)
    

Description of the dataset without negative restaurant distances
----------------------------------------------------------------

    df[df['avg_dist_rest'] > 0].describe().round(2)
    

*   The mean and standard deviation are not much affected by the removal of negative disatances
*   The above dataset will be use for further evaluations
*   The rows with negative values are thereby discarded

    df = df[df['avg_dist_rest'] > 0]
    df.describe().round(2)
    

Correlation
-----------

    df.corr()
    

*   It appears that there are no other important correlations other than order vs amount

    plt.figure(figsize=(20,10))
    plt.title('Number of orders vs Amount spent', fontsize=26)
    plt.xlabel('Number of Orders', fontsize=24)
    plt.ylabel('Amount Spent', fontsize=24)
    plt.scatter(df['no_of_orders'], df['amount'])
    plt.savefig('./plots/orders_vs_amount.png', format='png', dpi=1000)
    plt.show()
    

    plt.figure(figsize=(20,10))
    plt.title('Distribution of amount', fontsize=26)
    plt.ylabel('Amount Spent', fontsize=24)
    plt.hist(df['amount'], bins=[0, 10000, 20000, 30000, 40000, 50000, 60000, 70000])
    plt.savefig('./plots/distribution_amount.png', format='png', dpi=300)
    plt.show()
    

    plt.figure(figsize=(20,10))
    plt.title('Distribution of distances', fontsize=26)
    plt.ylabel('Average Distances', fontsize=24)
    plt.hist(df['avg_dist_rest'])
    plt.savefig('./plots/distribution_distances.png', format='png', dpi=300)
    plt.show()
    

    plt.figure(figsize=(20,10))
    plt.title('Distribution of delivery time', fontsize=26)
    plt.ylabel('Average Delivery Time', fontsize=24)
    plt.hist(df['avg_del_time'])
    plt.savefig('./plots/distribution_delivery.png', format='png', dpi=300)
    plt.show()
    

    df.head()
    

Calculating customer Recency
============================

    latest_recent_time = df['recent_time'].max()
    df['recency'] = df['recent_time'].apply(lambda x: (latest_recent_time - x).days)
    

Delivery score
--------------

*   A custom score given to every customer

    df['delivery_score'] = df['avg_del_time'] / df['avg_dist_rest']
    

    df.head()
    

    quantiles = df.quantile(q=[0.25,0.5,0.75])
    quantiles.to_dict()
    

    {'customer_id': {0.25: 335730.75, 0.5: 667378.0, 0.75: 1004865.75},
     'no_of_orders': {0.25: 1.0, 0.5: 2.0, 0.75: 7.0},
     'orders_7_days': {0.25: 1.0, 0.5: 1.0, 0.75: 2.0},
     'orders_last_4_weeks': {0.25: 1.0, 0.5: 2.0, 0.75: 4.0},
     'amount': {0.25: 280.0, 0.5: 691.0, 0.75: 2060.75},
     'amt_last_7_days': {0.25: 0.0, 0.5: 0.0, 0.75: 0.0},
     'amt_in_last_4_weeks': {0.25: 0.0, 0.5: 0.0, 0.75: 401.75},
     'avg_dist_rest': {0.25: 1.7, 0.5: 2.4, 0.75: 3.1},
     'avg_del_time': {0.25: 26.0, 0.5: 37.0, 0.75: 47.0},
     'recency': {0.25: 39.0, 0.5: 65.0, 0.75: 111.0},
     'delivery_score': {0.25: 10.74074074074074,
      0.5: 15.65217391304348,
      0.75: 22.5}}
    

    # Arguments (x = value, p = recency, monetary_value, frequency, d = quartiles dict)
    
    def RScore(x, p, d):
        if x <= d[p][0.25]:
            return 4
        elif x <= d[p][0.50]:
            return 3
        elif x <= d[p][0.75]: 
            return 2
        else:
            return 1
        
    # Arguments (x = value, p = recency, monetary_value, frequency, d = quartiles dict)
    def FMScore(x, p, d):
        if x <= d[p][0.25]:
            return 1
        elif x <= d[p][0.50]:
            return 2
        elif x <= d[p][0.75]: 
            return 3
        else:
            return 4
    

RFM Segemtation for last 7 days of customer orders
--------------------------------------------------

    rfm_segmentation_7_days = df
    rfm_segmentation_7_days['r_quartile_7_days'] = rfm_segmentation_7_days['recency'].apply(RScore, args=('recency',quantiles,))
    rfm_segmentation_7_days['f_quartile_7_days'] = rfm_segmentation_7_days['orders_7_days'].apply(FMScore, args=('orders_7_days',quantiles,))
    rfm_segmentation_7_days['m_quartile_7_days'] = rfm_segmentation_7_days['amt_last_7_days'].apply(FMScore, args=('amt_last_7_days',quantiles,))
    rfm_segmentation_7_days.head()
    

RFM Segemtation for last 4 weeks of customer orders
---------------------------------------------------

    rfm_segmentation_4_weeks = df
    rfm_segmentation_4_weeks['r_quartile_4_weeks'] = rfm_segmentation_4_weeks['recency'].apply(RScore, args=('recency',quantiles,))
    rfm_segmentation_4_weeks['f_quartile_4_weeks'] = rfm_segmentation_4_weeks['orders_last_4_weeks'].apply(FMScore, args=('orders_last_4_weeks',quantiles,))
    rfm_segmentation_4_weeks['m_quartile_4_weeks'] = rfm_segmentation_4_weeks['amt_in_last_4_weeks'].apply(FMScore, args=('amt_in_last_4_weeks',quantiles,))
    rfm_segmentation_4_weeks.head()
    

RFMScore = (r \* f \* m) \* delivery\_score
-------------------------------------------

    rfm_segmentation_7_days['RFMScore_7_days'] = rfm_segmentation_7_days.r_quartile_7_days \
                                * rfm_segmentation_7_days.f_quartile_7_days \
                                * rfm_segmentation_7_days.m_quartile_7_days \
                                * rfm_segmentation_7_days.delivery_score
    rfm_segmentation_7_days.head()
    

    rfm_segmentation_7_days.describe()
    

RFMScore = (r \* f \* m) \* delivery\_score
-------------------------------------------

    rfm_segmentation_4_weeks['RFMScore_4_weeks'] = rfm_segmentation_4_weeks.r_quartile_4_weeks \
                                * rfm_segmentation_4_weeks.f_quartile_4_weeks \
                                * rfm_segmentation_4_weeks.m_quartile_4_weeks \
                                * rfm_segmentation_4_weeks.delivery_score
    rfm_segmentation_4_weeks.head()
    

    rfm_segmentation_7_days.describe()['RFMScore_7_days']
    

    count     9934.000000
    mean       256.353922
    std        395.715899
    min         14.117647
    25%         83.478261
    50%        152.727273
    75%        271.562998
    max      11093.333333
    Name: RFMScore_7_days, dtype: float64
    

    rfm_segmentation_4_weeks.describe()['RFMScore_4_weeks']
    

    count     9934.000000
    mean       321.186736
    std        500.580048
    min         14.117647
    25%         82.666667
    50%        157.241379
    75%        355.555556
    max      12480.000000
    Name: RFMScore_4_weeks, dtype: float64
    

Moving on to K Means Clustering
-------------------------------

    rfm_segmentation_7_days_values = rfm_segmentation_7_days[['amt_last_7_days', 'RFMScore_7_days']].values
    rfm_segmentation_4_weeks_values = rfm_segmentation_4_weeks[['amt_in_last_4_weeks', 'RFMScore_4_weeks']].values
    

    wcss = []
    for i in range(1, 11):
        kmeans = KMeans(n_clusters=i, init='k-means++', max_iter=300, n_init=10, random_state=0)
        kmeans.fit(rfm_segmentation_7_days_values)
        wcss.append(kmeans.inertia_)
    
    plt.figure(figsize=(20,10))
    plt.plot(range(1, 11), wcss)
    plt.title('Finding the number of clusters - Elbow curve - last 7 days', fontsize=26)
    plt.xlabel('Number of clusters', fontsize=24)
    plt.ylabel('WCSS')
    plt.savefig('./plots/7_days_elbow.png', format='png', dpi=300)
    plt.show()
    

### The curve says that 2 would be the ideal number of clusters

    wcss = []
    for i in range(1, 11):
        kmeans = KMeans(n_clusters=i, init='k-means++', max_iter=300, n_init=10, random_state=0)
        kmeans.fit(rfm_segmentation_4_weeks_values)
        wcss.append(kmeans.inertia_)
    
    plt.figure(figsize=(20,10))
    plt.plot(range(1, 11), wcss)
    plt.title('Finding the number of clusters - Elbow curve - last 4 weeks', fontsize=26)
    plt.xlabel('Number of clusters', fontsize=24)
    plt.ylabel('WCSS')
    plt.savefig('./plots/4_weeks_elbow.png', format='png', dpi=300)
    plt.show()
    

### The curve says that 3 would be the ideal number of clusters

Applying K Means to customer data of past 7 days
------------------------------------------------

    kmeans = KMeans(n_clusters=2, init='k-means++', max_iter=300, n_init=10, random_state=0)
    y_kmeans = kmeans.fit_predict(rfm_segmentation_7_days_values)
    

    # Visualizing the clusters
    plt.figure(figsize=(20,10))
    plt.scatter(rfm_segmentation_7_days_values[y_kmeans==0, 0], rfm_segmentation_7_days_values[y_kmeans==0, 1], s=100, c='red', label='Cluster 1', alpha=0.5)
    plt.scatter(rfm_segmentation_7_days_values[y_kmeans==1, 0], rfm_segmentation_7_days_values[y_kmeans==1, 1], s=100, c='blue', label='Cluster 2', alpha=0.5)
    plt.scatter(kmeans.cluster_centers_[:, 0], kmeans.cluster_centers_[:, 1], s=300, c='yellow', label='Centroids')
    plt.title('Clusters of customers for last 7 days', fontsize=26)
    plt.xlabel('Amount', fontsize=24)
    plt.ylabel('RFMScore', fontsize=24)
    plt.legend(prop={'size': 30})
    plt.savefig('./plots/k_means_7_days.png', format='png', dpi=300)
    plt.show()
    

Applying K Means to customer data of past 4 weeks
-------------------------------------------------

    kmeans = KMeans(n_clusters=3, init='k-means++', max_iter=300, n_init=10, random_state=0)
    y_kmeans = kmeans.fit_predict(rfm_segmentation_4_weeks_values)
    
    # Visualizing the clusters
    plt.figure(figsize=(20,10))
    plt.scatter(rfm_segmentation_4_weeks_values[y_kmeans==0, 0], rfm_segmentation_4_weeks_values[y_kmeans==0, 1], s=100, c='red', label='Cluster 1', alpha=0.5)
    plt.scatter(rfm_segmentation_4_weeks_values[y_kmeans==1, 0], rfm_segmentation_4_weeks_values[y_kmeans==1, 1], s=100, c='blue', label='Cluster 2', alpha=0.5)
    plt.scatter(rfm_segmentation_4_weeks_values[y_kmeans==2, 0], rfm_segmentation_4_weeks_values[y_kmeans==2, 1], s=100, c='green', label='Cluster 3', alpha=0.5)
    
    plt.scatter(kmeans.cluster_centers_[:, 0], kmeans.cluster_centers_[:, 1], s=300, c='yellow', label='Centroids')
    plt.title('Clusters of customers for last 4 weeks', fontsize=26)
    plt.xlabel('Amount', fontsize=24)
    plt.ylabel('RFMScore', fontsize=24)
    plt.legend(prop={'size': 30})
    plt.savefig('./plots/k_means_4_weeks.png', format='png', dpi=300)
    plt.show()
    

Conclusion
==========

*   In a dataset of 10000 rows
    *   Recent orders (Last 7 days) can be categorized into two parts. One being least spending customers and the others are returning customers
    *   Past orders (Last 4 weeks) can be categorized into three parts. One being the least spending, another being loyal customers and the other being returning customers
*   The customers in least spending customers (Red Cluster) have more orders but less amount spent
*   The customers who are loyal (Green Cluster) are more like to spend money along with more orders
*   The customers who are returning (Blue Cluster) are more likely to spend money eventhough their order quantity is less

[Previous issue](https://sudhanva-narayana.ghost.io/cogeotiff-research-on-space-tech-saas-platform-at-pixxel/)

[Browse all issues](https://sudhanva-narayana.ghost.io/page/2)

[Next issue](https://sudhanva-narayana.ghost.io/install-kubeflow-on-digital-ocean-machine-learning/)