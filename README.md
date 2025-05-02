wget https://packages.wazuh.com/4.x/apt/pool/main/w/wazuh-agent/wazuh-agent_4.11.2-1_amd64.deb && sudo WAZUH_MANAGER='yd0k64kskz4f.cloud.wazuh.com' WAZUH_REGISTRATION_PASSWORD=$'6T8RVyd0MNYY9TVuSDqCSyCOvHpeVKYb' WAZUH_AGENT_GROUP='default' WAZUH_AGENT_NAME='test' dpkg -i ./wazuh-agent_4.11.2-1_amd64.deb

sudo systemctl daemon-reload
sudo systemctl enable wazuh-agent
sudo systemctl start wazuh-agent

https://yd0k64kskz4f.cloud.wazuh.com/app/endpoints-summary#/agents-preview/deploy
