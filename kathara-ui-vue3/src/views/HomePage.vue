<template>
  <ul class="breadcrumb">
    <li class="breadcrumb-item"><a href="#">Blockchain governance laboratory</a></li>
    <li class="breadcrumb-item active">Current Lab</li>
  </ul>
  <h1 class="page-header">
    {{ labStore.getLabName }} <small>{{ labStore.getLabDescription }}</small>
  </h1>
  <div class="mb-sm-4 mb-3 d-sm-flex">
    <div class="mt-sm-0 mt-2">
      <a href="#" class="text-inverse text-opacity-75 text-decoration-none" @click="exportGraphJson">
        <i class="fa fa-download fa-fw me-1 text-theme"></i> Export
      </a>
    </div>
    <div class="ms-sm-4 mt-sm-0 mt-2">
      <a href="#" class="text-inverse text-opacity-75 text-decoration-none"
         data-bs-toggle="modal"
         data-bs-target="#modalImportGraph">
        <i class="fa fa-upload fa-fw me-1 text-theme"></i> Import
      </a>
    </div>
    <div class="ms-sm-4 mt-sm-0 mt-2">
      <a
          href="#"
          class="text-inverse text-opacity-75 text-decoration-none {{ labState !== LabState.EDITING ? 'disabled': '' }}"
          @click="resetGraph"
      >
        <i class="fa fa-trash-alt fa-fw me-1 text-theme"></i> Reset
      </a>
    </div>
  </div>
  <div class="row gx-4">
    <div class="col-lg-6">
      <div class="card mb-4">
        <div class="card-header d-flex align-items-center bg-inverse bg-opacity-10 fw-400">
          Avilable nodes
        </div>
        <div class="card-body">
          <button
              type="button"
              class="btn btn-pink me-1 btn-lg"
              data-bs-toggle="modal"
              data-bs-target="#modalPeerNode"
              id="openPeerNode"
          >
            Add a peer node
          </button>
          <button
              type="button"
              class="btn btn-pink me-1 btn-lg"
              data-bs-toggle="modal"
              data-bs-target="#modalCaNode"
              id="openCaNode"
          >
            Add a ca node
          </button>
          <button
              type="button"
              class="btn btn-pink me-1 btn-lg"
              data-bs-toggle="modal"
              data-bs-target="#modalOrdererNode"
              id="openOrdererNode"
          >
            Add an orderer node
        </button>
        <button
              type="button"
              class="btn btn-pink me-1 btn-lg"
              data-bs-toggle="modal"
              data-bs-target="#modalChaincodeNode"
              id="openChaincodeNode"
          >
            Add a chaincode
        </button>
        </div>
        <div class="card-arrow">
          <div class="card-arrow-top-left"></div>
          <div class="card-arrow-top-right"></div>
          <div class="card-arrow-bottom-left"></div>
          <div class="card-arrow-bottom-right"></div>
        </div>
      </div>
    </div>
    <div class="col-lg-6">
      <div class="card mb-4">
        <div class="card-header d-flex align-items-center bg-inverse bg-opacity-10 fw-400">
          Deployment operations
        </div>
        <div class="card-body">
          <button
              type="button"
              class="btn btn-success me-1 btn-lg"
              @click="deployCas"
          >
            Deploy cas
          </button>
          <button
              type="button"
              class="btn btn-success me-1 btn-lg"
              @click="deployOrderers"
          >
            Deploy orderer
          </button>
          <button
              type="button"
              class="btn btn-success me-1 btn-lg"
              @click="deployPeers"
          >
            Deploy peers
          </button>
          <button
              type="button"
              class="btn btn-primary me-1 btn-lg"
              @click="enrollMSPIDS"
          >
            Enroll MSPIDs
          </button>
          <button
              type="button"
              class="btn btn-primary me-1 btn-lg"
              @click="createChannel"
          >
            Create channel
          </button>
          <button
              type="button"
              class="btn btn-primary me-1 btn-lg"
              @click="deployChaincode"
          >
            Deploy chaincode
          </button>
        </div>
        <div class="card-arrow">
          <div class="card-arrow-top-left"></div>
          <div class="card-arrow-top-right"></div>
          <div class="card-arrow-bottom-left"></div>
          <div class="card-arrow-bottom-right"></div>
        </div>
      </div>
    </div>
  </div>
  <v-network-graph
    class="graph"
    v-model:selected-nodes="selectedNodes"
    v-model:selected-edges="selectedEdges"
    :nodes="nodes"
    :edges="edges"
    :layouts="layout"
    :configs="configs"
    :event-handlers="eventHandlers"
  >
    <defs>
      <!--
        Define the path for clipping the face image.
        To change the size of the applied node as it changes,
        add the `clipPathUnits="objectBoundingBox"` attribute
        and specify the relative size (0.0~1.0).
      -->
      <clipPath id="iconCircle" clipPathUnits="objectBoundingBox">
        <circle cx="0.5" cy="0.5" r="0.5" />
      </clipPath>
    </defs>
    <!-- Replace the node component -->
    <template #override-node="{ nodeId, scale, config, ...slotProps }">
      <!-- circle for filling background -->
      <circle
        class="icon-circle"
        :r="config.radius * scale"
        fill="#ffffff"
        v-bind="slotProps"
      />
      <!--
        The base position of the <image /> is top left. The node's
        center should be (0,0), so slide it by specifying x and y.
      -->
      <image
        class="icon-picture"
        :x="-config.radius * scale"
        :y="-config.radius * scale"
        :width="config.radius * scale * 2"
        :height="config.radius * scale * 2"
        :xlink:href="`./icons/${nodes[nodeId].icon}`"
        clip-path="url(#iconCircle)"
      />
      <!-- circle for drawing stroke -->
      <circle
        class="icon-circle"
        :r="config.radius * scale"
        fill="none"
        stroke="#808080"
        :stroke-width="1 * scale"
        v-bind="slotProps"
      />
    </template>
  </v-network-graph>
  <!-- BEGIN #PeerNode -->
  <div class="modal fade" id="modalPeerNode">
    <div class="modal-dialog">
      <div class="modal-content">
        <div class="modal-header">
          <h5 class="modal-title">Add peer</h5>
          <button
            type="button"
            class="btn-close"
            data-bs-dismiss="modal"
            @click="closePeerNodeModal"
          ></button>
        </div>
        <div class="modal-body">
          <div class="mb-3">
            <label class="form-label">MSPID:</label>
            <div class="row row-space-10">
              <div class="col-12">
                <input
                  class="form-control"
                  type="text"
                  placeholder="e.g. Org1MSP"
                  v-model="mspidmodel"
                 
                />
                <!---  @keypress="isCollisionDomainName($event)" -->
              </div>
            </div>
          </div>
        </div>
        <div class="modal-body">
          <div class="mb-3">
            <label class="form-label">Name:</label>
            <div class="row row-space-10">
              <div class="col-12">
                <input
                  class="form-control"
                  type="text"
                  placeholder="e.g. org1-peer0"
                  v-model="namemodel"
                />
                <!---  @keypress="isCollisionDomainName($event)" -->
              </div>
            </div>
          </div>
        </div>
        <div class="modal-body">
          <div class="mb-3">
            <label class="form-label">Capacity:</label>
            <div class="row row-space-10">
              <div class="col-12">
                <input
                  class="form-control"
                  type="text"
                  placeholder="e.g. 5Gi"
                  v-model="capmodel"
                 
                />
                <!---  @keypress="isCollisionDomainName($event)" -->
              </div>
            </div>
          </div>
        </div>
        <div class="modal-body">
          <div class="mb-3">
            <label class="form-label">Ca name:</label>
            <div class="row row-space-10">
              <div class="col-12">
                <input
                  class="form-control"
                  type="text"
                  placeholder="e.g. org1-ca.default"
                  v-model="canamemodel"
                 
                />
                <!---  @keypress="isCollisionDomainName($event)" -->
              </div>
            </div>
          </div>
        </div>
        <div class="modal-body">
          <div class="mb-3">
            <label class="form-label">hosts:</label>
            <div class="row row-space-10">
              <div class="col-12">
                <input
                  class="form-control"
                  type="text"
                  placeholder="e.g. peer0-org1.localho.st"
                  v-model="hostmodel"
                 
                />
                <!---  @keypress="isCollisionDomainName($event)" -->
              </div>
            </div>
          </div>
        </div>
        <div class="modal-body">
          <div class="mb-3">
            <label class="form-label">Istio port:</label>
            <div class="row row-space-10">
              <div class="col-12">
                <input
                  class="form-control"
                  type="text"
                  placeholder="e.g. 443"
                  v-model="istioportmodel"
                 
                />
                <!---  @keypress="isCollisionDomainName($event)" -->
              </div>
            </div>
          </div>
        </div>
        <div class="modal-body">
          <div class="mb-3">
            <label class="form-label">Namespace:</label>
            <div class="row row-space-10">
              <div class="col-12">
                <input
                  class="form-control"
                  type="text"
                  placeholder="e.g. default"
                  v-model="namespacemodel"
                 
                />
                <!---  @keypress="isCollisionDomainName($event)" -->
              </div>
            </div>
          </div>
        </div>
        <div class="modal-footer">
          <button
            type="button"
            class="btn btn-outline-default"
            data-bs-dismiss="modal"
            id="closeModalPeerNode"
            @click="closePeerNodeModal"
          >
            Close
          </button>
          <button
            type="button"
            class="btn btn-outline-theme"
            @click="addEditPeerNode"
          >
            Save changes
          </button>
        </div>
      </div>
    </div>
  </div>

  <!-- begin #modalCaNode -->
  <div class="modal fade" id="modalCaNode">
    <div class="modal-dialog">
      <div class="modal-content">
        <div class="modal-header">
          <h5 class="modal-title">Add CA</h5>
          <button
            type="button"
            class="btn-close"
            data-bs-dismiss="modal"
            @click="closeCaNodeModal"
          ></button>
        </div>
        <div class="modal-body">
          <div class="mb-3">
            <label class="form-label">MSPID:</label>
            <div class="row row-space-10">
              <div class="col-12">
                <input
                  class="form-control"
                  type="text"
                  placeholder="e.g. Org1MSP"
                  v-model="camspidmodel"
                 
                />
                <!---  @keypress="isCollisionDomainName($event)" -->
              </div>
            </div>
          </div>
        </div>
        <div class="modal-body">
          <div class="mb-3">
            <label class="form-label">Name:</label>
            <div class="row row-space-10">
              <div class="col-12">
                <input
                  class="form-control"
                  type="text"
                  placeholder="e.g. org1-ca"
                  v-model="cacanamemodel"
                />
                <!---  @keypress="isCollisionDomainName($event)" -->
              </div>
            </div>
          </div>
        </div>
        <div class="modal-body">
          <div class="mb-3">
            <label class="form-label">Capacity:</label>
            <div class="row row-space-10">
              <div class="col-12">
                <input
                  class="form-control"
                  type="text"
                  placeholder="e.g. 1Gi"
                  v-model="cacapmodel"
                 
                />
                <!---  @keypress="isCollisionDomainName($event)" -->
              </div>
            </div>
          </div>
        </div>
        <div class="modal-body">
          <div class="mb-3">
            <label class="form-label">hosts:</label>
            <div class="row row-space-10">
              <div class="col-12">
                <input
                  class="form-control"
                  type="text"
                  placeholder="e.g. org1-ca.default"
                  v-model="cahostmodel"
                 
                />
                <!---  @keypress="isCollisionDomainName($event)" -->
              </div>
            </div>
          </div>
        </div>
        <div class="modal-body">
          <div class="mb-3">
            <label class="form-label">Istio port:</label>
            <div class="row row-space-10">
              <div class="col-12">
                <input
                  class="form-control"
                  type="text"
                  placeholder="e.g. 443"
                  v-model="caistioportmodel"
                 
                />
                <!---  @keypress="isCollisionDomainName($event)" -->
              </div>
            </div>
          </div>
        </div>
        <div class="modal-body">
          <div class="mb-3">
            <label class="form-label">Namespace:</label>
            <div class="row row-space-10">
              <div class="col-12">
                <input
                  class="form-control"
                  type="text"
                  placeholder="e.g. default"
                  v-model="canamespacemodel"
                 
                />
                <!---  @keypress="isCollisionDomainName($event)" -->
              </div>
            </div>
          </div>
        </div>
        <div class="modal-footer">
          <button
            type="button"
            class="btn btn-outline-default"
            data-bs-dismiss="modal"
            id="closeModalCaNode"
            @click="closeCaNodeModal"
          >
            Close
          </button>
          <button
            type="button"
            class="btn btn-outline-theme"
            @click="addEditCaNode"
          >
            Save changes
          </button>
        </div>
      </div>
    </div>
  </div>
  <!-- begin #modalOrdererNode -->
  <div class="modal fade" id="modalOrdererNode">
    <div class="modal-dialog">
      <div class="modal-content">
        <div class="modal-header">
          <h5 class="modal-title">Add Orderer</h5>
          <button
            type="button"
            class="btn-close"
            data-bs-dismiss="modal"
            @click="closeOrdererNodeModal"
          ></button>
        </div>
        <div class="modal-body">
          <div class="mb-3">
            <label class="form-label">MSPID:</label>
            <div class="row row-space-10">
              <div class="col-12">
                <input
                  class="form-control"
                  type="text"
                  placeholder="e.g. Org1MSP"
                  v-model="orderermspidmodel"
                 
                />
                <!---  @keypress="isCollisionDomainName($event)" -->
              </div>
            </div>
          </div>
        </div>
        <div class="modal-body">
          <div class="mb-3">
            <label class="form-label">Name:</label>
            <div class="row row-space-10">
              <div class="col-12">
                <input
                  class="form-control"
                  type="text"
                  placeholder="e.g. ord-node1"
                  v-model="orderernamemodel"
                />
                <!---  @keypress="isCollisionDomainName($event)" -->
              </div>
            </div>
          </div>
        </div>
        <div class="modal-body">
          <div class="mb-3">
            <label class="form-label">Capacity:</label>
            <div class="row row-space-10">
              <div class="col-12">
                <input
                  class="form-control"
                  type="text"
                  placeholder="e.g. 2Gi"
                  v-model="orderercapmodel"
                 
                />
                <!---  @keypress="isCollisionDomainName($event)" -->
              </div>
            </div>
          </div>
        </div>
        <div class="modal-body">
          <div class="mb-3">
            <label class="form-label">caname:</label>
            <div class="row row-space-10">
              <div class="col-12">
                <input
                  class="form-control"
                  type="text"
                  placeholder="e.g. org1-ca.default"
                  v-model="ordererhostmodel"
                 
                />
                <!---  @keypress="isCollisionDomainName($event)" -->
              </div>
            </div>
          </div>
        </div>
        <div class="modal-body">
          <div class="mb-3">
            <label class="form-label">Istio port:</label>
            <div class="row row-space-10">
              <div class="col-12">
                <input
                  class="form-control"
                  type="text"
                  placeholder="e.g. 443"
                  v-model="ordereristioportmodel"
                 
                />
                <!---  @keypress="isCollisionDomainName($event)" -->
              </div>
            </div>
          </div>
        </div>
        <div class="modal-body">
          <div class="mb-3">
            <label class="form-label">Endpoint:</label>
            <div class="row row-space-10">
              <div class="col-12">
                <input
                  class="form-control"
                  type="text"
                  placeholder="e.g. 443"
                  v-model="ordererendpointmodel"
                 
                />
                <!---  @keypress="isCollisionDomainName($event)" -->
              </div>
            </div>
          </div>
        </div>
        <div class="modal-body">
          <div class="mb-3">
            <label class="form-label">External port:</label>
            <div class="row row-space-10">
              <div class="col-12">
                <input
                  class="form-control"
                  type="text"
                  placeholder="e.g. 443"
                  v-model="ordererexternalportmodel"
                 
                />
                <!---  @keypress="isCollisionDomainName($event)" -->
              </div>
            </div>
          </div>
        </div>
        <div class="modal-body">
          <div class="mb-3">
            <label class="form-label">Port:</label>
            <div class="row row-space-10">
              <div class="col-12">
                <input
                  class="form-control"
                  type="text"
                  placeholder="e.g. 443"
                  v-model="ordererportmodel"
                 
                />
                <!---  @keypress="isCollisionDomainName($event)" -->
              </div>
            </div>
          </div>
        </div>
        <div class="modal-body">
          <div class="mb-3">
            <label class="form-label">Namespace:</label>
            <div class="row row-space-10">
              <div class="col-12">
                <input
                  class="form-control"
                  type="text"
                  placeholder="e.g. default"
                  v-model="orderernamespacemodel"
                 
                />
                <!---  @keypress="isCollisionDomainName($event)" -->
              </div>
            </div>
          </div>
        </div>
        <div class="modal-footer">
          <button
            type="button"
            class="btn btn-outline-default"
            data-bs-dismiss="modal"
            id="closeModalOrdererNode"
            @click="closeOrdererNodeModal"
          >
            Close
          </button>
          <button
            type="button"
            class="btn btn-outline-theme"
            @click="addEditOrdererNode"
          >
            Save changes
          </button>
        </div>
      </div>
    </div>
  </div>


  <!-- begin #modalCaNode -->
  <div class="modal fade" id="modalChaincodeNode">
    <div class="modal-dialog">
      <div class="modal-content">
        <div class="modal-header">
          <h5 class="modal-title">Add Chaincode</h5>
          <button
            type="button"
            class="btn-close"
            data-bs-dismiss="modal"
            @click="closeChaincodeNodeModal"
          ></button>
        </div>
        <div class="modal-body">
          <div class="mb-3">
            <label class="form-label">Name:</label>
            <div class="row row-space-10">
              <div class="col-12">
                <input
                  class="form-control"
                  type="text"
                  placeholder="e.g. token"
                  v-model="ccnamemodel"
                 
                />
                <!---  @keypress="isCollisionDomainName($event)" -->
              </div>
            </div>
          </div>
        </div>
        <div class="modal-body">
          <div class="mb-3">
            <label class="form-label">DockerUrl:</label>
            <div class="row row-space-10">
              <div class="col-12">
                <input
                  class="form-control"
                  type="text"
                  placeholder="e.g. janzumbier/tokencontract"
                  v-model="ccdockerurlmodel"
                />
                <!---  @keypress="isCollisionDomainName($event)" -->
              </div>
            </div>
          </div>
        </div>
        <div class="modal-body">
          <div class="mb-3">
            <label class="form-label">Policy:</label>
            <div class="row row-space-10">
              <div class="col-12">
                <input
                  class="form-control"
                  type="text"
                  placeholder="e.g. OR('ORG1MSP')"
                  v-model="ccpolicymodel"
                 
                />
                <!---  @keypress="isCollisionDomainName($event)" -->
              </div>
            </div>
          </div>
        </div>
        <div class="modal-body">
          <div class="mb-3">
            <label class="form-label">Version</label>
            <div class="row row-space-10">
              <div class="col-12">
                <input
                  class="form-control"
                  type="text"
                  placeholder="e.g. 1.0"
                  v-model="ccversionmodel"
                 
                />
                <!---  @keypress="isCollisionDomainName($event)" -->
              </div>
            </div>
          </div>
        </div>
        <div class="modal-body">
          <div class="mb-3">
            <label class="form-label">Sequence:</label>
            <div class="row row-space-10">
              <div class="col-12">
                <input
                  class="form-control"
                  type="text"
                  placeholder="e.g. 1"
                  v-model="ccsequencemodel"
                />
                <!---  @keypress="isCollisionDomainName($event)" -->
              </div>
            </div>
          </div>
        </div>
        <div class="modal-body">
          <div class="mb-3">
            <label class="form-label">Namespace:</label>
            <div class="row row-space-10">
              <div class="col-12">
                <input
                  class="form-control"
                  type="text"
                  placeholder="e.g. default"
                  v-model="ccnamespacemodel"
                 
                />
                <!---  @keypress="isCollisionDomainName($event)" -->
              </div>
            </div>
          </div>
        </div>
        <div class="modal-footer">
          <button
            type="button"
            class="btn btn-outline-default"
            data-bs-dismiss="modal"
            id="closeModalChaincodeNode"
            @click="closeChaincodeNodeModal"
          >
            Close
          </button>
          <button
            type="button"
            class="btn btn-outline-theme"
            @click="addEditChaincodeNode"
          >
            Save changes
          </button>
        </div>
      </div>
    </div>
  </div>

  


  <div class="modal fade" id="modalNetworkDevice">
    <div class="modal-dialog">
      <div class="modal-content">
        <div class="modal-header">
          <h5 class="modal-title">Add/Edit Network Device</h5>
          <button
            type="button"
            class="btn-close"
            data-bs-dismiss="modal"
            @click="closeNetworkDeviceModal"
          ></button>
        </div>
        <div class="modal-body">
          <div class="mb-3">
            <label class="form-label">Device Type (*):</label>
            <div class="row row-space-10">
              <div class="col-12">
                <select
                  v-model="selectedDeviceType"
                  class="form-select-lg"
                  @change="onDeviceTypeChange"
                  :disabled="!nodeMode"
                >
                  <option disabled value="">Please select one</option>
                  <option value="client">PC Client</option>
                  <option value="web-server">Web Server</option>
                  <option value="linux-switch">Linux Switch</option>
                  <option value="traefik-balancer">Traefik Balancer</option>
                  <option value="dns-server">DNS Server</option>
                  <option value="vpn-device">VPN (Client/Server)</option>
                  <option value="vyatta-router">Vyatta Router</option>
                  <option value="frr-router">FRR Router</option>
                </select>
              </div>
            </div>
          </div>
          <div class="mb-3">
            <label class="form-label">Device Name (*):</label>
            <div class="row row-space-10">
              <div class="col-12">
                <input
                  class="form-control"
                  type="text"
                  placeholder="Device name (e.g., pc1, linux_switch...)"
                  v-model="deviceName"
                  @keypress="isDeviceName($event)"
                />
              </div>
            </div>
          </div>
          <div class="mb-3">
            <label class="form-label">Device Image (*):</label>
            <div class="row row-space-10">
              <div class="col-12">
                <input
                  class="form-control"
                  type="text"
                  v-model="deviceDockerImage"
                />
              </div>
            </div>
          </div>
          <div class="mb-3">
            <label class="form-label">Device Startup Script:</label>
            <div class="row row-space-10">
              <div class="col-12">
                <textarea
                  class="form-control"
                  v-model="deviceStartupScript"
                  rows="10"
                ></textarea>
              </div>
            </div>
          </div>
          <div class="mb-3">
            <label class="form-label">Device Shutdown Script:</label>
            <div class="row row-space-10">
              <div class="col-12">
                <textarea
                    class="form-control"
                    v-model="deviceShutdownScript"
                    rows="10"
                ></textarea>
              </div>
            </div>
          </div>
          <div class="mb-3">
            <label class="form-label">Device Memory:</label>
            <div class="row row-space-10">
              <div class="col-12">
                <input
                  class="form-control"
                  type="text"
                  placeholder="Allocated memory for device. Format: xxx[b/k/m/g] (e.g., 128m, 1g ...)"
                  v-model="deviceMemory"
                />
              </div>
            </div>
          </div>
          <div class="mb-3">
            <label class="form-label">Device CPUs:</label>
            <div class="row row-space-10">
              <div class="col-12">
                <input
                  class="form-control"
                  type="text"
                  placeholder="Allocated CPU shares (e.g., 0.5, 1.5, 2...)."
                  v-model="deviceCPUShare"
                />
              </div>
            </div>
          </div>
          <div class="mb-3">
            <label class="form-label">Device Bridged:</label>
            <div class="row row-space-10">
              <div class="col-12">
                <div class="form-check form-switch">
                  <input
                    type="checkbox"
                    class="form-check-input"
                    id="deviceBridgeEnabled"
                    v-model="deviceBridgeEnabled"
                  />
                  <label class="form-check-label" for="deviceBridgeEnabled">
                    Connect device to host?
                  </label>
                </div>
              </div>
            </div>
          </div>
          <div class="mb-3">
            <label class="form-label">Support IPv6:</label>
            <div class="row row-space-10">
              <div class="col-12">
                <div class="form-check form-switch">
                  <input
                    type="checkbox"
                    class="form-check-input"
                    id="deviceIPv6Enabled"
                    v-model="deviceIPv6Enabled"
                  />
                  <label class="form-check-label" for="deviceIPv6Enabled">
                    Device supports IPv6
                  </label>
                </div>
              </div>
            </div>
          </div>
          <div class="mb-3">
            <label class="form-label">Device shell cmds:</label>
            <div class="row row-space-10">
              <div class="col-12">
                <textarea
                  class="form-control"
                  v-model="deviceExecCmds"
                  rows="3"
                ></textarea>
              </div>
            </div>
          </div>
          <div class="mb-3">
            <label class="form-label">Device sysctl options:</label>
            <div class="row row-space-10">
              <div class="col-12">
                <textarea
                  class="form-control"
                  v-model="deviceSysctlOptions"
                  rows="3"
                ></textarea>
              </div>
            </div>
          </div>
          <div class="mb-3">
            <label class="form-label">Device envs:</label>
            <div class="row row-space-10">
              <div class="col-12">
                <textarea
                  class="form-control"
                  v-model="deviceEnv"
                  rows="3"
                ></textarea>
              </div>
            </div>
          </div>
          <div class="mb-3">
            <label class="form-label">Device shell:</label>
            <div class="row row-space-10">
              <div class="col-12">
                <input
                  class="form-control"
                  type="text"
                  placeholder="Default shell used when connecting to device (e.g., bash, sh)"
                  v-model="deviceShell"
                />
              </div>
            </div>
          </div>
          <div class="alert alert-muted">
            <b>Note:</b>
            Device name must be unique. Kathara cannot have two devices with the
            same name. Fields with (*) are mandatory. Otherwise, they are
            optional.
          </div>
        </div>
        <div class="modal-footer">
          <button
            type="button"
            class="btn btn-outline-default"
            data-bs-dismiss="modal"
            id="closeModalNetworkDevice"
            @click="closeNetworkDeviceModal"
          >
            Close
          </button>
          <button
            type="button"
            class="btn btn-outline-theme"
            :disabled="!isDeviceNameEligible(deviceName)"
            @click="addEditNetworkDevice"
          >
            Save changes
          </button>
        </div>
      </div>
    </div>
  </div>
  <!-- END #modalNetworkDevice -->
  <!-- BEGIN #modalEdge -->
  <div class="modal fade" id="modalEdge">
    <div class="modal-dialog">
      <div class="modal-content">
        <div class="modal-header">
          <h5 class="modal-title">Add/Edit Link</h5>
          <button
            type="button"
            class="btn-close"
            data-bs-dismiss="modal"
          ></button>
        </div>
        <div class="modal-body">
          <div class="mb-3">
            <label class="form-label">Eth:</label>
            <div class="row row-space-10">
              <div class="col-12">
                <input
                  class="form-control"
                  type="text"
                  placeholder="Interface Index (e.g., 0, 1, 2 ...)"
                  v-model="selectedDeviceInterfaceIndex"
                  @keypress="isNumber($event)"
                />
              </div>
            </div>
          </div>
          <div class="mb-3">
            <label class="form-label">Collision Domain Code:</label>
            <div class="row row-space-10">
              <div class="col-12">
                <input
                  class="form-control"
                  type="text"
                  v-model="selectedCDValue"
                  readonly
                />
              </div>
            </div>
          </div>
        </div>
        <div class="modal-footer">
          <button
            type="button"
            class="btn btn-outline-default"
            data-bs-dismiss="modal"
            id="closeModalEdge"
          >
            Close
          </button>
          <button
            type="button"
            class="btn btn-outline-theme"
            @click="addEditEdge"
          >
            Save changes
          </button>
        </div>
      </div>
    </div>
  </div>
  <!-- END #modalEdge -->
  <!-- BEGIN #modalImportGraph -->
  <div class="modal fade" id="modalImportGraph">
    <div class="modal-dialog">
      <div class="modal-content">
        <div class="modal-header">
          <h5 class="modal-title">Import graph from JSON</h5>
          <button
              type="button"
              class="btn-close"
              data-bs-dismiss="modal"
          ></button>
        </div>
        <div class="modal-body">
          <div class="mb-3">
            <label class="form-label">File:</label>
            <div class="row row-space-10">
              <div class="col-12">
                <input
                    class="form-control"
                    type="file"
                    ref="jsonFile"
                    accept="application/JSON"
                    @change="onFileChange"
                />
              </div>
            </div>
          </div>
        </div>
        <div class="modal-footer">
          <button
              type="button"
              class="btn btn-outline-default"
              data-bs-dismiss="modal"
              id="closeModalImportGraph"
          >
            Close
          </button>
          <button
              type="button"
              class="btn btn-outline-theme"
              @click="importGraphFromJSON"
          >
            Import
          </button>
        </div>
      </div>
    </div>
  </div>
  <!-- END #modalImportGraph -->
  <!-- BEGIN #labToast -->
  <div class="toasts-container">
    <div :class="['toast', 'text-black', toastType === 0 ? 'bg-info' : 'bg-danger']" data-autohide="false" id="lab-toast">
      <div class="d-flex">
        <div class="toast-body">
          {{ toastMessage }}
        </div>
        <button type="button" class="btn-close btn-close-white me-2 m-auto" data-bs-dismiss="toast" aria-label="Close"></button>
      </div>
    </div>
  </div>
  <!-- END #labToast -->
  <!--
    BEGIN #draggableConsole
    Note: must use key & id = machine_name so components can be correctly removed
  -->
  <div class="console-view">
    <component
        v-for="(item, idx) of consoleIframeComponents"
        :key="item.machine_name"
        :id="item.machine_name"
        :is="mapTypeComponents[item.component_name]"
        :machine_name="item.machine_name"
        @closeConsoleFrame="closeTTy"
    />
  </div>
  <!-- END #draggableConsole -->
</template>

<script setup lang="ts">
import {defineAsyncComponent, onMounted, reactive, ref, watch} from "vue";
import {storeToRefs} from "pinia";
import {useGraphStore} from "@/stores/app-graph";
import {useLabStore} from "@/stores/app-lab";
import {Toast} from "bootstrap";
import type {CaNode, ChaincodeNode, CollisionDomain, DeviceInterface, NetworkDevice, OrdererNode, PeerNode, TopologyModel} from "@/models/graph-models";

import * as vNG from "v-network-graph";
import {VNetworkGraph} from "v-network-graph";

// json
import FileSaver, { saveAs } from "file-saver";
import { HTMLInputEvent } from "@/models/extra-models";
const jsonFile = ref<InstanceType<typeof HTMLInputElement>>();
const jsonContent = ref("");

// worker
import {LabState} from "@/models/lab-states";
import type {ConsoleIframe} from "@/models/lab-models";

// get pinia stores
const graphStore = useGraphStore();
const labStore = useLabStore();

// v-network-graph variables
const { nodes, edges, usedCdCodes, nextEdgeIndex, layout } = storeToRefs(graphStore);

const configs = reactive(
  vNG.defineConfigs({
    node: {
      selectable: 2, // up to 2 nodes
      normal: {
        radius: 20,
      },
      hover: {
        radius: 22,
      },
      label: {
        visible: true,
        fontSize: 14,
        lineHeight: 1.1,
        color: "#000",
        margin: 4,
        direction: "south",
        text: "name",
      },
    },
    edge: {
      selectable: 1,
      normal: {
        width: 1,
        color: "#2222aa",
      },
      hover: {
        width: 2,
        color: "#0000aa",
      },
      gap: 20,
      type: "curve",
      margin: 6,
      marker: {
        source: {
          type: "circle",
          width: 8,
          height: 8,
        },
        target: {
          type: "circle",
          width: 8,
          height: 8,
        },
      },
    },
  })
);

// onMounted
onMounted(() => {
  console.log("REST link:" + import.meta.env.VITE_KATHARA_API_URL);
  console.log("TTY link:" + import.meta.env.VITE_WEBTTY_API_URL);
})

// lab variables
const { currentState: labState, visibleConsoleFrames } = storeToRefs(labStore);

watch(labState, async (value, oldValue) => {
  if (value !== oldValue) {
    if (oldValue === LabState.EDITING && value === LabState.CREATED) {
      showToast("Successfully created lab...", 0);
    }
    if (oldValue === LabState.CREATED && value === LabState.STARTING) {
      showToast("Lab is starting...", 0);
    }

    if (oldValue === LabState.STARTING && value === LabState.RUNNING) {
      showToast("Lab is running...", 0);
    }

    if (oldValue === LabState.RUNNING && value === LabState.CLEANING) {
      showToast("Lab is being cleaned...", 1);
    }

    if (oldValue === LabState.CLEANING && value === LabState.REMOVED) {
      showToast("Lab is successfully wiped...", 1);
      await labStore.resetLabState();
    }
  }
})

// toast variables
const toastMessage = ref("");
const toastType = ref(0);

// iframe variables
const consoleIframeComponents = ref<ConsoleIframe[]>([]);
const mapTypeComponents = {
  draggableConsole: defineAsyncComponent(() => import("@/components/custom/DraggableConsole.vue"))
}

const eventHandlers: vNG.EventHandlers = {
  "node:dragend": (draggedNode) => {
    const draggedNodeId = Object.keys(draggedNode)[0];
    console.log(
      `Drag node ${draggedNodeId} with x-pos: ${JSON.stringify(
        draggedNode[draggedNodeId].x
      )} and y-pos: ${JSON.stringify(draggedNode[draggedNodeId].y)}`
    );
    graphStore.updateNodePosition(draggedNodeId,
      draggedNode[draggedNodeId].x,
      draggedNode[draggedNodeId].y);
    //nodes[draggedNodeId].pos_X = draggedNode[draggedNodeId].x;
    //nodes[draggedNodeId].pos_Y = draggedNode[draggedNodeId].y;
  },
  "node:dblclick": (clickedNode) => {
    if (clickedNode.event.detail == 2) {
      //console.log(`dblclick on ${JSON.stringify(clickedNode)}`)
      // node can only be edited when lab is in EDITING mode
      if (labState.value === LabState.EDITING &&
          nodes.value[clickedNode.node].node_type === "network_device") {
        nodeMode.value = false;
        editedNode.value = clickedNode.node;
        selectedDeviceType.value = nodes.value[clickedNode.node].type;
        deviceName.value = nodes.value[clickedNode.node].name!;
        deviceDockerImage.value = nodes.value[clickedNode.node].docker_image;
        deviceStartupScript.value = nodes.value[clickedNode.node].startup_script;
        deviceShutdownScript.value= nodes.value[clickedNode.node].shutdown_script;
        deviceMemory.value = nodes.value[clickedNode.node].memory;
        deviceCPUShare.value = nodes.value[clickedNode.node].cpus;
        deviceBridgeEnabled.value = nodes.value[clickedNode.node].bridged;
        deviceIPv6Enabled.value = nodes.value[clickedNode.node].ipv6;
        deviceExecCmds.value = nodes.value[clickedNode.node].exec;
        deviceSysctlOptions.value = nodes.value[clickedNode.node].sysctl;
        deviceEnv.value = nodes.value[clickedNode.node].env;
        deviceShell.value = nodes.value[clickedNode.node].shell;
        //console.log(`editedNode = ${editedNode.value}`);
        document.getElementById("openNetworkDeviceModal")!.click();
      }

      if(labState.value === LabState.EDITING &&
      nodes.value[clickedNode.node].node_type === "peer_node"){
        nodeMode.value = false;
        editedNode.value = clickedNode.node;
        namemodel.value = nodes.value[clickedNode.node].name;
        mspidmodel.value = nodes.value[clickedNode.node].mspid;
        canamemodel.value = nodes.value[clickedNode.node].caname;
        namespacemodel.value = nodes.value[clickedNode.node].namespace;
        hostmodel.value = nodes.value[clickedNode.node].hosts;
        capmodel.value = nodes.value[clickedNode.node].capacity;
        istioportmodel.value = nodes.value[clickedNode.node].istioport;
        document.getElementById("openPeerNode")!.click();
      }

      if(labState.value === LabState.EDITING &&
      nodes.value[clickedNode.node].node_type === "ca_node"){
        nodeMode.value = false;
        editedNode.value = clickedNode.node;
        camspidmodel.value = nodes.value[clickedNode.node].mspid;
        cahostmodel.value  = nodes.value[clickedNode.node].hosts; 
        cacanamemodel.value = nodes.value[clickedNode.node].name; 
        caistioportmodel.value = nodes.value[clickedNode.node].istioport;
        cacapmodel.value = nodes.value[clickedNode.node].capacity;
        canamespacemodel.value = nodes.value[clickedNode.node].namespace;
        document.getElementById("openCaNode")!.click();
      }

      if(labState.value === LabState.EDITING &&
      nodes.value[clickedNode.node].node_type === "orderer_node"){
        nodeMode.value = false;
        editedNode.value = clickedNode.node;
        orderermspidmodel.value = nodes.value[clickedNode.node].mspid; 
        ordererhostmodel.value = nodes.value[clickedNode.node].hosts;
        orderernamemodel.value = nodes.value[clickedNode.node].name
        orderercanamemodel.value = nodes.value[clickedNode.node].caname 
        ordereristioportmodel.value = nodes.value[clickedNode.node].istioport
        orderercapmodel.value = nodes.value[clickedNode.node].capacity
        orderernamespacemodel.value = nodes.value[clickedNode.node].namespace
        ordererportmodel.value = nodes.value[clickedNode.node].port
        ordererexternalportmodel.value = nodes.value[clickedNode.node].externalport
        ordererendpointmodel.value = nodes.value[clickedNode.node].endpoint
        document.getElementById("openOrdererNode")!.click();
      }

      if(labState.value === LabState.EDITING &&
      nodes.value[clickedNode.node].node_type === "chaincode_node"){
        nodeMode.value = false;
        editedNode.value = clickedNode.node;
        ccnamemodel.value = nodes.value[clickedNode.node].name;
        ccdockerurlmodel.value = nodes.value[clickedNode.node].dockerurl;
        ccversionmodel.value = nodes.value[clickedNode.node].version;
        ccsequencemodel.value = nodes.value[clickedNode.node].sequence;
        cclanguagemodel.value = nodes.value[clickedNode.node].language;
        ccpolicymodel.value = nodes.value[clickedNode.node].policy;
        ccnamespacemodel.value = nodes.value[clickedNode.node].namespace;
        document.getElementById("openChaincodeNode")!.click(); 
      }



      // node web-tty iframe can only be opened when lab is in RUNNING mode.
      if (labState.value === LabState.RUNNING) {
        if (!labStore.checkConsoleIframeVisibility(clickedNode.node)) {
          console.log(`Open machine ${clickedNode.node} web-tty iframe`);
          labStore.showMachineConsoleIframe(clickedNode.node);
          consoleIframeComponents.value.push({
            component_name: "draggableConsole",
            machine_name: clickedNode.node,
          })
        }
      }
    }
  },
  /*"node:click": (clickedNode) => {
    if (
      clickedNode.event.detail === 1 &&
      nodes[clickedNode.node].type === "collision_domain"
    ) {
      selectedCDValue.value = nodes[clickedNode.node].code;
      console.log(`Update CD value to ${selectedCDValue.value}`);
    }
  },*/
  "node:select": (selectedNodes) => {
    if (selectedNodes.length === 0) return;
    // update the selectedCD
    if (
      selectedNodes.length === 1 &&
        nodes.value[selectedNodes[0]].node_type === "collision_domain"
    ) {
      selectedCDValue.value = nodes.value[selectedNodes[0]].code;
      //console.log(`Update CD value to ${selectedCDValue.value}`);
    }
    if (selectedNodes.length === 2) {
      if (
          nodes.value[selectedNodes[0]].node_type !== nodes.value[selectedNodes[1]].node_type
      ) {
        selectedCDValue.value =
            nodes.value[selectedNodes[0]].node_type === "collision_domain"
            ? nodes.value[selectedNodes[0]].code
            : nodes.value[selectedNodes[1]].code;
        //console.log(`Update CD value to ${selectedCDValue.value}`);
      }
    }
  },
  "edge:select": (selectedEdges) => {
    if (selectedEdges.length === 0) return;
    //console.log(`Select the edge: ${JSON.stringify(selectedEdges)}`);
    edgeMode = false;
    selectedDeviceInterfaceIndex.value = edges.value[selectedEdges[0]].info.index;
    selectedCDValue.value = edges.value[selectedEdges[0]].info.cd;
  },
};

// v-network-graph system
const selectedNodes = ref<string[]>([]);
const selectedEdges = ref<string[]>([]);
/*const nextCDIndex = ref(
  Object.keys(nodes).filter((n) => n.indexOf("cd") >= 0).length + 1
);
const nextNDIndex = ref(
  Object.keys(nodes).filter((n) => n.indexOf("nd") >= 0).length + 1
);*/
const selectedDeviceType = ref("");

// used CD codes
const newCdCode = ref("");

// new network device properties
const nodeMode = ref(true); // true => add new cd/device, false => edit dbl cd/device
const editedNode = ref("");
const deviceName = ref("");
//
const mspidmodel = ref("");
const namemodel = ref("");
const capmodel = ref("");
const istioportmodel = ref("");
const canamemodel = ref("");
const hostmodel = ref("");
const namespacemodel = ref("");
//
const camspidmodel = ref("");
const cacanamemodel = ref("");
const caistioportmodel = ref("");
const cacapmodel = ref("");
const cahostmodel = ref("");
const canamespacemodel = ref("");
//

const orderernamemodel = ref("");
const orderermspidmodel = ref("");
const orderercapmodel = ref("");
const ordererhostmodel = ref("");
const ordereristioportmodel = ref("");
const ordererendpointmodel = ref("");
const orderernamespacemodel = ref("");
const ordererportmodel = ref("");
const ordererexternalportmodel = ref("");
const orderercanamemodel = ref("");

//

const ccnamemodel = ref("");
const ccdockerurlmodel = ref("");
const ccnamespacemodel = ref("");
const ccpolicymodel = ref("");
const ccversionmodel = ref("");
const ccsequencemodel = ref("");
const cclanguagemodel = ref("");




const deviceDockerImage = ref("");
const deviceStartupScript = ref("");
const deviceShutdownScript = ref("");
const deviceMemory = ref("");
const deviceCPUShare = ref(1);
const deviceBridgeEnabled = ref(false);
const deviceIPv6Enabled = ref(false);
const deviceExecCmds = ref("");
const deviceSysctlOptions = ref("");
const deviceEnv = ref("");
const deviceShell = ref("");

// node functions
const isCollisionDomainName = (e: KeyboardEvent) => {
  const char = e.key;
  if (/^[A-Z]*$/.test(char)) return true;
  else e.preventDefault();
};

const isDeviceName = (e: KeyboardEvent) => {
  const char = e.key;
  if (/^[A-Za-z0-9_]+$/.test(char)) return true;
  else e.preventDefault();
};

const isCdCodeEligible = (newCode: string): boolean => {
  if (newCode === "") return false;
  return !usedCdCodes.value.some((code) => code === newCode);
};

const isDeviceNameEligible = (newName: string): boolean => {
  if (!nodeMode.value) return true;
  if (newName === "" && nodeMode) return false;
  return !(Object.keys(nodes) as Array<string>).some(
    (deviceName) => deviceName === newName
  );
};

const isNumber = (e: KeyboardEvent) => {
  const char = e.key;
  if (/^[0-9]+$/.test(char)) return true;
  else e.preventDefault();
};


const addEditChaincodeNode = () => {
  if(nodeMode.value) {
    const newChaincodeNode: ChaincodeNode = {
      node_type: "chaincode_node",
      icon: "collision-domain.png",
      name: ccnamemodel.value,
      version: ccversionmodel.value,
      sequence: ccsequencemodel.value,
      dockerurl: ccdockerurlmodel.value,
      language: cclanguagemodel.value,
      policy: ccpolicymodel.value,
      namespace: ccnamespacemodel.value
    }
    const nodeId = newChaincodeNode.name;
    nodes.value[nodeId] = newChaincodeNode;
    document.getElementById("closeModalChaincodeNode")!.click();
  } else {
    nodes.value[editedNode.value].name = ccnamemodel.value
    nodes.value[editedNode.value].dockerurl = ccdockerurlmodel.value
    nodes.value[editedNode.value].version = ccversionmodel.value
    nodes.value[editedNode.value].sequence = ccsequencemodel.value
    nodes.value[editedNode.value].language= cclanguagemodel.value
    nodes.value[editedNode.value].policy = ccpolicymodel.value
    nodes.value[editedNode.value].namespace = ccnamespacemodel.value
    document.getElementById("closeModalChaincodeNode")!.click();
  }
}

const closeChaincodeNodeModal = () => {
  nodeMode.value = true;
  ccnamemodel.value = "";
  ccdockerurlmodel.value = "";
  ccsequencemodel.value = "";
  ccversionmodel.value = "";
  cclanguagemodel.value = "";
  ccpolicymodel.value = "";
  ccnamespacemodel.value = "";
  editedNode.value = "";
};

const addEditOrdererNode = () => {
  if(nodeMode.value) {
    const newOrdererNode: OrdererNode = {
      node_type: "orderer_node",
      mspid: orderermspidmodel.value,
      icon: "collision-domain.png",
      name: orderernamemodel.value,
      caname: orderercanamemodel.value,
      namespace: orderernamespacemodel.value,
      hosts: ordererhostmodel.value,
      capacity: orderercapmodel.value,
      istioport: ordereristioportmodel.value,
      port: ordererportmodel.value,
      endpoint: ordererendpointmodel.value,
      externalport: ordererexternalportmodel.value
    }
    const nodeId = newOrdererNode.name;
    nodes.value[nodeId] = newOrdererNode;
    document.getElementById("closeModalOrdererNode")!.click();
  } else {

    nodes.value[editedNode.value].mspid = orderermspidmodel.value
    nodes.value[editedNode.value].hosts = ordererhostmodel.value
    nodes.value[editedNode.value].name = orderernamemodel.value
    nodes.value[editedNode.value].caname = orderercanamemodel.value
    nodes.value[editedNode.value].istioport= ordereristioportmodel.value
    nodes.value[editedNode.value].capacity = orderercapmodel.value
    nodes.value[editedNode.value].namespace = orderernamespacemodel.value
    nodes.value[editedNode.value].port = ordererportmodel.value
    nodes.value[editedNode.value].externalport = ordererexternalportmodel.value
    nodes.value[editedNode.value].endpoint = ordererendpointmodel.value
    document.getElementById("closeModalOrdererNode")!.click();
  }
}

const closeOrdererNodeModal = () => {
  nodeMode.value = true;
  orderermspidmodel.value = "";
  orderernamemodel.value = "";
  orderercanamemodel.value = "";
  orderernamespacemodel.value = "";
  ordererhostmodel.value = "";
  orderercapmodel.value= "";
  ordereristioportmodel.value = "";
  ordererportmodel.value = "";
  ordererendpointmodel.value = "";
  ordererexternalportmodel.value = "";
  editedNode.value = "";
};

const addEditCaNode = () => {
  if(nodeMode.value) {
    const newCaNode: CaNode = {
      node_type: "ca_node",
      mspid: camspidmodel.value,
      icon: "collision-domain.png",
      name: cacanamemodel.value,
      namespace: canamespacemodel.value,
      hosts: cahostmodel.value,
      capacity: cacapmodel.value,
      istioport: caistioportmodel.value
    }
    const nodeId = newCaNode.name;
    nodes.value[nodeId] = newCaNode;
    document.getElementById("closeModalCaNode")!.click();
  } else {

    nodes.value[editedNode.value].mspid = camspidmodel.value
    nodes.value[editedNode.value].hosts = cahostmodel.value
    nodes.value[editedNode.value].name = cacanamemodel.value
    nodes.value[editedNode.value].istioport= caistioportmodel.value
    nodes.value[editedNode.value].capacity = cacapmodel.value
    nodes.value[editedNode.value].namespace = canamespacemodel.value
    document.getElementById("closeModalCaNode")!.click();
  }
}

const closeCaNodeModal = () => {
  nodeMode.value = true;
  camspidmodel.value = "";
  cacanamemodel.value = "";
  cahostmodel.value = "";
  canamespacemodel.value = "";
  cacapmodel.value = "";
  caistioportmodel.value = "";
  editedNode.value = "";
};



const addEditPeerNode = () => {
  if (nodeMode.value) {
    const newPeerNode: PeerNode = {
      node_type: "peer_node",
      mspid: mspidmodel.value,
      icon: "collision-domain.png",
      name: namemodel.value,
      caname: canamemodel.value,
      namespace: namespacemodel.value,
      hosts: hostmodel.value,
      capacity: capmodel.value,
      istioport: istioportmodel.value
    }
    const nodeId = newPeerNode.name;
    nodes.value[nodeId] = newPeerNode;
    document.getElementById("closeModalPeerNode")!.click();
  } else {
    nodes.value[editedNode.value].mspid = mspidmodel.value
    nodes.value[editedNode.value].hosts = hostmodel.value
    nodes.value[editedNode.value].name = namemodel.value
    nodes.value[editedNode.value].caname = canamemodel.value
    nodes.value[editedNode.value].istioport= istioportmodel.value
    nodes.value[editedNode.value].capacity = capmodel.value
    nodes.value[editedNode.value].namespace = namespacemodel.value
    document.getElementById("closeModalPeerNode")!.click();
  }

}

const addEditCollisionDomain = () => {
  if (nodeMode.value) {
    const newCollisionDomain: CollisionDomain = {
      code: newCdCode.value,
      node_type: "collision_domain",
      name: newCdCode.value,
      icon: "collision-domain.png"
      // pos_X: 100,
      // pos_Y: 100,
    };
    const nodeId = `cd_${newCdCode.value}`;
    nodes.value[nodeId] = newCollisionDomain;
    usedCdCodes.value.push(newCdCode.value);
    document.getElementById("closeModalCollisionDomain")!.click();
  }
};

const closePeerNodeModal = () => {
  nodeMode.value = true;
  mspidmodel.value = "";
  namemodel.value = "";
  canamemodel.value = "";
  hostmodel.value = "";
  namespacemodel.value = "";
  capmodel.value = "";
  istioportmodel.value = "";
  editedNode.value = "";
};

function deviceIcon(deviceType: string): string {
  switch (deviceType) {
    case "linux-switch":
      return "network-switch.png";
    case "traefik-balancer":
      return "network-balancer.png";
    case "dns-server":
      return "network-dns.png";
    case "vyatta-router":
    case "frr-router":
      return "network-router.png";
    default:
      return "network-pc.png";
  }
}

const addEditNetworkDevice = () => {
  if (nodeMode.value) {
    const newNetworkDevice: NetworkDevice = {
      name: deviceName.value,
      node_type: "network_device",
      type: selectedDeviceType.value,
      docker_image: deviceDockerImage.value,
      icon: deviceIcon(selectedDeviceType.value),
      interfaces: [],
      startup_script:
        deviceStartupScript.value !== ""
          ? deviceStartupScript.value
          : undefined,
      shutdown_script:
          deviceShutdownScript.value !== ""
              ? deviceShutdownScript.value
              : undefined,
      memory: deviceMemory.value !== "" ? deviceMemory.value : undefined,
      cpus: deviceCPUShare.value !== 1 ? deviceCPUShare.value : undefined,
      bridged: deviceBridgeEnabled.value,
      ipv6: deviceIPv6Enabled.value,
      exec: deviceExecCmds.value !== "" ? deviceExecCmds.value : undefined,
      sysctl:
        deviceSysctlOptions.value !== ""
          ? deviceSysctlOptions.value
          : undefined,
      env: deviceEnv.value !== "" ? deviceEnv.value : undefined,
      shell: deviceShell.value !== "" ? deviceShell.value : undefined,
      // pos_X: 100,
      // pos_Y: 100,
    };
    const nodeId = deviceName.value;
    nodes.value[nodeId] = newNetworkDevice;
    // add to layout
    layout.value.nodes[nodeId] = {
      x: 100,
      y: 100
    }
  } else {
    nodes.value[editedNode.value].docker_image = deviceDockerImage.value;
    nodes.value[editedNode.value].startup_script =
      deviceStartupScript.value !== "" ? deviceStartupScript.value : undefined;
    nodes.value[editedNode.value].shutdown_script =
        deviceShutdownScript.value !== "" ? deviceShutdownScript.value : undefined;
    nodes.value[editedNode.value].memory =
      deviceMemory.value !== "" ? deviceMemory.value : undefined;
    nodes.value[editedNode.value].cpus =
      deviceCPUShare.value !== 1 ? deviceCPUShare.value : undefined;
    nodes.value[editedNode.value].bridged = deviceBridgeEnabled.value;
    nodes.value[editedNode.value].ipv6 = deviceIPv6Enabled.value;
    nodes.value[editedNode.value].exec =
      deviceExecCmds.value !== "" ? deviceExecCmds.value : undefined;
    nodes.value[editedNode.value].sysctl =
      deviceSysctlOptions.value !== "" ? deviceSysctlOptions.value : undefined;
    nodes.value[editedNode.value].env =
      deviceEnv.value !== "" ? deviceEnv.value : undefined;
    nodes.value[editedNode.value].shell =
      deviceShell.value !== "" ? deviceShell.value : undefined;
  }
  document.getElementById("closeModalNetworkDevice")!.click();
};

const closeNetworkDeviceModal = () => {
  // reset value
  deviceName.value = "";
  deviceMemory.value = "";
  deviceCPUShare.value = 1;
  deviceBridgeEnabled.value = false;
  deviceIPv6Enabled.value = false;
  deviceStartupScript.value = "";
  deviceShutdownScript.value = "";
  deviceExecCmds.value = "";
  deviceSysctlOptions.value = "";
  deviceEnv.value = "";
  deviceShell.value = "";
  nodeMode.value = true;
  editedNode.value = "";
};

// edge functions
let edgeMode = false; // true => add new edge, false => edit selected edge
const selectedDeviceInterfaceIndex = ref("0");
const selectedCDValue = ref("");

const isEdgeEligible = (selected: string[]): boolean => {
  if (selected.length !== 2) return false;
  if (labState.value !== LabState.EDITING) return false;
  const sourceName = selected[0];
  const targetName = selected[1];
  return nodes.value[sourceName].node_type !== nodes.value[targetName].node_type;
};

const openEdgeModal = (mode: boolean) => {
  edgeMode = mode;
};

const addEditEdge = () => {
  // create new link between device and CD
  if (edgeMode) {
    const [sourceName, targetName] = selectedNodes.value;
    let newDeviceInterface: DeviceInterface;
    // add new interface to device
    if (nodes.value[sourceName].node_type === "network_device") {
      newDeviceInterface = {
        index: selectedDeviceInterfaceIndex.value,
        cd: nodes.value[targetName].code,
      };
      nodes.value[sourceName].interfaces.push(newDeviceInterface);
    } else {
      newDeviceInterface = {
        index: selectedDeviceInterfaceIndex.value,
        cd: nodes.value[sourceName].code,
      };
      nodes.value[targetName].interfaces.push(newDeviceInterface);
    }
    // add edge to graph
    nextEdgeIndex.value++;
    const edgeId = `edge${nextEdgeIndex.value}`;
    edges.value[edgeId] = {
      source: sourceName,
      target: targetName,
      info: newDeviceInterface,
    };
  } else {
    // edit the current selected link
    console.log(`Edit the link ${edges.value[selectedEdges.value[0]]}`);
  }

  // close modal
  document.getElementById("closeModalEdge")!.click();
};

const removeEdge = () => {
  const edgeId = selectedEdges.value[0];
  const sourceNode = nodes.value[edges.value[edgeId].source];
  const targetNode = nodes.value[edges.value[edgeId].target];
  if (sourceNode.type === "network_device") {
    const cd_value = targetNode.code;
    sourceNode.interfaces = sourceNode.interfaces.filter(
      (int: DeviceInterface) => {
        return int.cd !== cd_value;
      }
    );
  } else {
    const cd_value = sourceNode.code;
    targetNode.interfaces = targetNode.interfaces.filter(
      (int: DeviceInterface) => {
        return int.cd !== cd_value;
      }
    );
  }
  delete edges.value[edgeId];
};

const onDeviceTypeChange = () => {
  switch (selectedDeviceType.value) {
    case "web-server":
      deviceDockerImage.value = "unibaktr/alpine:whoami";
      deviceSysctlOptions.value = "";
      deviceShell.value = "";
      break;
    case "traefik-balancer":
      deviceDockerImage.value = "unibaktr/alpine:traefik";
      deviceSysctlOptions.value = "";
      deviceShell.value = "";
      break;
    case "dns-server":
      deviceDockerImage.value = "unibaktr/alpine:coredns";
      deviceSysctlOptions.value = "";
      deviceShell.value = "";
      break;
    case "vpn-device":
      deviceDockerImage.value = "unibaktr/alpine:wireguard";
      deviceSysctlOptions.value = "";
      deviceShell.value = "";
      break;
    case "vyatta-router":
      deviceDockerImage.value = "unibaktr/vyos:1.4";
      deviceSysctlOptions.value = "net.ipv6.conf.all.disable_ipv6=0";
      deviceShell.value = "/bin/vyos";
      break;
    case "frr-router":
      deviceDockerImage.value = "unibaktr/alpine:frr";
      deviceSysctlOptions.value = "";
      deviceShell.value = "";
      break;
    case "client":
    case "linux-switch":
      deviceDockerImage.value = "unibaktr/alpine";
      deviceSysctlOptions.value = "";
      deviceShell.value = "";
      break;
    default:
      deviceDockerImage.value = "unibaktr/alpine";
      deviceSysctlOptions.value = "";
      deviceShell.value = "";
      break;
  }
};

const showToast = (message: string, type: number) => {
  toastMessage.value = message;
  toastType.value = type;
  const toast = new Toast(document.getElementById("lab-toast")!);
  toast.show();
}

const closeTTy = (machineName: string) => {
  console.log("Trying to close tty of machine=" + machineName)
  labStore.hideMachineConsoleIframe(machineName);
  const frameIdx = consoleIframeComponents.value.findIndex(c => c.machine_name === machineName);
  console.log("Found index=" + frameIdx);
  consoleIframeComponents.value.splice(frameIdx, 1);
}

const exportGraphJson = () => {
  if (Object.keys(nodes.value).length > 0) {
    const graphModel: TopologyModel = {
      nodes: nodes.value,
      edges: edges.value,
      usedCdCodes: usedCdCodes.value,
      nextEdgeIndex: nextEdgeIndex.value,
      layout: layout.value,
    };

    const exportLab = {
      name: labStore.getLabName,
      description: labStore.getLabDescription,
      version: labStore.getLabVersion,
      email: labStore.getLabEmail,
      author: labStore.getLabAuthor,
      model: graphModel,
    }

    const graphJSONFile = new File([JSON.stringify(exportLab)],
        labStore.getLabName + ".json",
        {type: "text/plain;charset=utf-8"})
    FileSaver.saveAs(graphJSONFile);
    showToast("Export lab successfully.", 0);
  } else {
    showToast("Cannot export lab. Add a least one node to do so.", 1);
  }
};

function onFileChange(event: HTMLInputEvent) {
  let files = (event as HTMLInputEvent).target.files;
  if (!files?.length) return
  // show for debug
  const fileReader = new FileReader();
  fileReader.onload = (res) => {
    jsonContent.value = <string>res.target?.result;
  }
  fileReader.onerror = (err) => {
    console.log(err)
  }
  fileReader.readAsText(files[0]);
}

function importGraphFromJSON() {
  const graphObj = JSON.parse(jsonContent.value);
  // import lab information
  labStore.importLab(graphObj);
  // import graph information
  graphStore.importGraph(graphObj);

  // reset file input
  jsonFile.value!.value = "";

  // close modal
  document.getElementById("closeModalImportGraph")!.click();

  // show toast

}

const resetGraph = () => {
  graphStore.resetGraph();
}

const createLab = async () => {
  console.log("CREATING laboratory")
  labStore.convertGraphToTopo(nodes.value);
  // myWorker.send("Hello worker!").then((reply: any) => console.log(reply));
  await labStore.createLab();
}

const runLab = async () => {
  console.log("RUNNING lab");
  await labStore.runLab();
}

const deployCas = async () => {
  showToast("Starting CAs deployment", 0);
  await labStore.getdeploycas();
  showToast("CAs deployed", 0);

}

const deployPeers = async () => {
  showToast("Starting peer deployment", 0);
  await labStore.getdeploypeers();
  showToast("Peers deployed", 0);
  console.log("finished");
}

const deployOrderers = async () => {
  showToast("Starting Orderer deployment", 0);
  await labStore.getdeployorderers();
  showToast("Orderers deployed", 0);
  console.log("finished");
}

const enrollMSPIDS = async () => {
  showToast("Enrolling MSPID", 0);
  await labStore.getenrollmspid();
  showToast("MSPIDs enrolled", 0);
}

const createChannel = async () => {
  showToast("Creating channel", 0);
  await labStore.getcreatechannel();
  showToast("Channel created", 0);
}

const deployChaincode = async () => {
  showToast("Starting chaincode deployment", 0);
  await labStore.getdeploychaincode();
  showToast("Chaincode deployed", 0);
}

const stopLab = async () => {
  for (let machineName of visibleConsoleFrames.value) {
    const frameIdx = consoleIframeComponents.value.findIndex(c => c.machine_name === machineName);
    consoleIframeComponents.value.splice(frameIdx, 1);
  }
  labStore.removeAllMachineConsoleIframe();
  await labStore.stopLab();
}

</script>

<style scoped>
.graph {
  width: 100%;
  height: 600px;
  border: 1px solid #fff;
  background-color: aliceblue;
}
.icon-circle,
.icon-picture {
  transition: all 0.1s linear;
}

.icon-picture {
  pointer-events: none;
}
.disabled {
  pointer-events: none;
  cursor: default;
}
.console-view {
  position: fixed;
  top: 0;
  left: 0;
}
</style>
