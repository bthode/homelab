kubectl create namespace kafka
kubectl create -f 'https://strimzi.io/install/latest?namespace=kafka' -n kafka

kubectl apply -f https://raw.githubusercontent.com/strimzi/strimzi-kafka-operator/0.34.0/examples/kafka/kafka-ephemeral-single.yaml -n kafka 
