---
title: 简单使用 Hyperledger-Fabric-Java-SDK
date: 2020-03-11 18:12:06
categories: 技术分享
tags:
- Java
- Fabric
---



介绍如何用 `Fabric Java SDK` 进行简单的数据块插入和查询
# 环境
- Hyperledger Fabric `2.0`
- Fabric Java SDK `2.0.0`
 
# 示例
```java
public class LocalUser implements User {
    private String name;
    private String mspId;
    private Enrollment enrollment;

    LocalUser(String name, String mspId, String keyFile, String certFile) throws Exception {
        this.name = name;
        this.mspId = mspId;
        this.enrollment = loadFromPemFile(keyFile, certFile);
    }

    private Enrollment loadFromPemFile(String keyFile, String certFile) throws Exception {
        byte[] keyPem = Files.readAllBytes(Paths.get(keyFile));
        byte[] certPem = Files.readAllBytes(Paths.get(certFile));
        CryptoPrimitives suite = new CryptoPrimitives();
        PrivateKey privateKey = suite.bytesToPrivateKey(keyPem);
        return new X509Enrollment(privateKey, new String(certPem));
    }

    @Override
    public String getName() {
        return name;
    }

    public Set<String> getRoles() {
        return null;
    }

    public String getMspId() {
        return mspId;
    }

    @Override
    public Enrollment getEnrollment() {
        return enrollment;
    }

    @Override
    public String getAccount() {
        return null;
    }

    @Override
    public String getAffiliation() {
        return null;
    }

}
```

```java
public class FabricTest {

    private static final String BASE_PATH = "./fabric";
    private static final String CHANNEL_NAME = "CHANNEL_NAME";
    private static final String CHAINCODE_ID = "CHAINCODE_ID";
    private static final Map<String, String> ORDERER_MAP = new HashMap<>();
    private static final Map<String, String> PEER_MAP = new HashMap<>();
    private static final boolean TLS = true;

    static
        String protocol = "grpc";
        if(TLS) {
            protocol += "s";
        }
        ORDERER_MAP.put("test.com", protocol + "://127.0.0.1:7050");
        PEER_MAP.put("peer.test.com", protocol + "://127.0.0.1:7051");
    }

    public static void main(String[] args) throws Exception {
        String keyFile = BASE_PATH + "***/user-key.pem";
        String certFile = BASE_PATH + "***/user-cert.pem";
        LocalUser user = new LocalUser("username", "mspId", keyFile, certFile);

        HFClient client = HFClient.createNewInstance();
        client.setCryptoSuite(CryptoSuite.Factory.getCryptoSuite());
        client.setUserContext(user);

        Channel channel = client.newChannel(CHANNEL_NAME);
        for (Map.Entry<String, String> entry : ORDERER_MAP.entrySet()) {
            Properties properties = new Properties();
            if(TLS) {
                properties.setProperty("clientCertFile", "对应配置目录下users/***/tls/client.crt文件");
                properties.setProperty("clientKeyFile", "对应配置目录下users/***/tls/client.key文件");
                properties.setProperty("pemFile", "对应配置目录下/tls/server.crt文件");
                properties.setProperty("hostnameOverride", entry.getKey());
                properties.setProperty("sslProvider", "openSSL");
                properties.setProperty("negotiationType", "TLS");
            }
            channel.addOrderer(client.newOrderer(entry.getKey(), entry.getValue(), properties));
        }
        for (Map.Entry<String, String> entry : PEER_MAP.entrySet()) {
            Properties properties = new Properties();
            if(TLS) {
                properties.setProperty("clientCertFile", "对应配置目录下users/***/tls/client.crt文件");
                properties.setProperty("clientKeyFile", "对应配置目录下users/***/tls/client.key文件");
                properties.setProperty("pemFile", "对应配置目录下/tls/server.crt文件");
                properties.setProperty("hostnameOverride", entry.getKey());
                properties.setProperty("sslProvider", "openSSL");
                properties.setProperty("negotiationType", "TLS");
            }
            channel.addPeer(client.newPeer(entry.getKey(), entry.getValue(), properties));
        }
        channel.initialize();

        System.out.format("Valid: %s\n", invokeRequest(client, channel, "insert", "ID1","VALUE222"));
        System.out.format("Message: %s\n", queryRequest(client, channel, "query", "ID1"));
    }

    private static String queryRequest(HFClient client, Channel channel, String function, String... args) throws Exception {
        QueryByChaincodeRequest request = client.newQueryProposalRequest();
        request.setChaincodeName(CHAINCODE_ID);
        request.setFcn(function);
        request.setArgs(args);
        ProposalResponse[] responses = channel.queryByChaincode(request).toArray(new ProposalResponse[0]);
        return responses[0].getProposalResponse().getResponse().getPayload().toStringUtf8();
    }

    private static boolean invokeRequest(HFClient client, Channel channel, String function, String... args) throws Exception {
        TransactionProposalRequest request = client.newTransactionProposalRequest();
        request.setChaincodeName(CHAINCODE_ID);
        request.setFcn(function);
        request.setArgs(args);
        Collection<ProposalResponse> responses = channel.sendTransactionProposal(request);
        BlockEvent.TransactionEvent event = channel.sendTransaction(responses).get();
        return event.isValid();
    }
}
```
